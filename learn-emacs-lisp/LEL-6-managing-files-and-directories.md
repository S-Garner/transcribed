# Learn Emacs Lisp 6: Managing Files and Directories

Emacs has excellent interactive tools for files: `find-file`, Dired, Eshell, and many more.

Eventually, though, you will want to automate file work with Emacs Lisp. That might mean generating files, moving configuration into a dotfiles repo, creating directories before writing output, or linking files into the places where other programs expect them.

This tutorial covers:

- `default-directory`
- manipulating file paths
- resolving relative and absolute paths
- checking whether files and directories exist
- creating directories
- listing files recursively
- copying, moving, and deleting files
- creating symbolic links
- using these tools to finish a small dotfiles manager

The practical example continues the dotfiles package from the previous lessons. The goal is to store configuration files in a dotfiles repository and create symbolic links into the user's home directory.

## What Is a Symbolic Link?

A symbolic link is a filesystem entry that points somewhere else.

For dotfiles, this is the usual idea:

```text
~/.dotfiles/files/.config/geeks
```

contains the real configuration, while:

```text
~/.config/geeks
```

is a symbolic link pointing to it.

Programs read `~/.config/geeks` as if it were a normal file or directory, but the real contents live in the dotfiles repository.

This lets you keep configuration in Git while still making it appear in the locations programs expect.

## The Project Goal

So far, the dotfiles helper can:

- tangle Org configuration files
- find tangled output paths
- update `.gitignore`

Now it needs to handle ordinary configuration files too.

Not every file belongs inside an Org source block. Some configuration files are plain data, application-specific formats, images, scripts, or whole directories. For those, the package needs to mirror a folder from the dotfiles repo into the home directory using symbolic links.

The examples below use the `dotfiles-` prefix to match the earlier tutorials in this repo.

## `default-directory`

`default-directory` is the current directory for a buffer.

Evaluate it in a file buffer:

```elisp
default-directory
```

In a normal file buffer, it is usually the directory containing that file.

In special buffers like `*scratch*`, it may be the directory where Emacs started, or whatever another package has set it to.

This matters because many file functions resolve relative paths against `default-directory`.

For package code, prefer passing explicit base directories:

```elisp
(expand-file-name "emacs.org" dotfiles-folder)
```

That is clearer than relying on the current buffer's directory.

## File Path Pieces

Emacs has many functions for splitting file paths apart.

Assume the current buffer is visiting:

```text
/home/user/notes/streams/emacs-lisp-06.org
```

`buffer-file-name` returns the full path:

```elisp
(buffer-file-name)
;; => "/home/user/notes/streams/emacs-lisp-06.org"
```

Get the directory part:

```elisp
(file-name-directory (buffer-file-name))
;; => "/home/user/notes/streams/"
```

Get the non-directory part:

```elisp
(file-name-nondirectory (buffer-file-name))
;; => "emacs-lisp-06.org"
```

Get the extension:

```elisp
(file-name-extension (buffer-file-name))
;; => "org"
```

Get the path without its extension:

```elisp
(file-name-sans-extension (buffer-file-name))
;; => "/home/user/notes/streams/emacs-lisp-06"
```

Get the base name without directory or extension:

```elisp
(file-name-base (buffer-file-name))
;; => "emacs-lisp-06"
```

Turn a path into a directory-style path:

```elisp
(file-name-as-directory
 (file-name-sans-extension (buffer-file-name)))
;; => "/home/user/notes/streams/emacs-lisp-06/"
```

That trailing slash matters. Some file functions distinguish a directory path from a file path by whether the string ends in `/`.

## Absolute and Relative Paths

Use `file-name-absolute-p` to test whether a path is absolute:

```elisp
(file-name-absolute-p "/home/user/.emacs.d/init.el")
;; => t

(file-name-absolute-p "init.el")
;; => nil

(file-name-absolute-p ".config/geeks")
;; => nil
```

On Unix-like systems, absolute paths start with `/`.

On Windows, an absolute path may start with a drive name, such as:

```text
C:/Users/example
```

## Relative Names

`file-relative-name` computes a path relative to another directory.

```elisp
(file-relative-name
 "/home/user/notes/streams/emacs-lisp-06.org"
 "/home/user/notes/")
;; => "streams/emacs-lisp-06.org"
```

If the file is not under the base directory, the relative path may include `..`:

```elisp
(file-relative-name
 "/home/user/notes/streams/emacs-lisp-06.org"
 "/home/user/.dotfiles/")
;; => "../notes/streams/emacs-lisp-06.org"
```

That is useful for detecting whether a path is inside the directory you expected.

## Expand File Names

Use `expand-file-name` to turn a relative path into an absolute path.

With one argument, it expands relative to `default-directory`:

```elisp
(expand-file-name "init.el")
```

With two arguments, it expands relative to a specific directory:

```elisp
(expand-file-name "init.el" "~/.emacs.d/")
;; => "/home/user/.emacs.d/init.el"
```

The target does not need to exist. `expand-file-name` is path arithmetic, not an existence check.

## Environment Variables in File Names

`expand-file-name` does not substitute environment variables inside strings:

```elisp
(expand-file-name "$HOME/.emacs.d")
;; => ".../$HOME/.emacs.d"
```

Use `substitute-in-file-name` when you want `$HOME` or similar variables expanded:

```elisp
(substitute-in-file-name "$HOME/.emacs.d")
;; => "/home/user/.emacs.d"
```

## Dotfiles Path Variables

The package needs to know:

- where the dotfiles repo lives
- where files should be linked
- which subdirectory contains linkable configuration files
- which output directories must exist before linking

```elisp
(defcustom dotfiles-folder "~/.dotfiles"
  "The path to the dotfiles folder."
  :type 'string
  :group 'dotfiles)

(defcustom dotfiles-output-directory "~"
  "Directory where dotfiles should be linked."
  :type 'string
  :group 'dotfiles)

(defcustom dotfiles-config-files-directory "files"
  "Directory inside `dotfiles-folder' containing files to link."
  :type 'string
  :group 'dotfiles)

(defcustom dotfiles-ensure-output-directories
  '(".config" ".local/share")
  "Directories to create in `dotfiles-output-directory' before linking."
  :type '(repeat string)
  :group 'dotfiles)
```

`dotfiles-output-directory` is usually the home directory.

It is useful to make it configurable because tests and demos should not create links directly in your real home directory.

## Resolve the Config Files Directory

The config files directory is stored as a path relative to the dotfiles repo:

```text
files
```

Resolve it to an absolute path:

```elisp
(defun dotfiles--config-files-path ()
  "Return the absolute path to the dotfiles config files directory."
  (expand-file-name dotfiles-config-files-directory
                    dotfiles-folder))
```

This gives the package one place to change the path logic.

## Resolve a Config File Target

Given a file inside:

```text
~/.dotfiles/files/.config/geeks/config.scm
```

the target path should be:

```text
~/.config/geeks/config.scm
```

The process is:

1. Find the file's path relative to the config files directory.
2. Expand that relative path against the output directory.

```elisp
(defun dotfiles--config-file-target (config-file)
  "Return the target path for CONFIG-FILE in `dotfiles-output-directory'."
  (expand-file-name
   (file-relative-name
    (expand-file-name config-file)
    (dotfiles--config-files-path))
   dotfiles-output-directory))
```

Example:

```elisp
(dotfiles--config-file-target
 "~/.dotfiles/files/.emacs.d/init.el")
;; => "/home/user/.emacs.d/init.el"
```

The exact result depends on your `dotfiles-folder`, `dotfiles-config-files-directory`, and `dotfiles-output-directory`.

## Check Whether Files Exist

Use `file-exists-p` to check whether a file or directory exists:

```elisp
(file-exists-p "~/.emacs.d")
;; => t

(file-exists-p "~/.emacs.d/nope")
;; => nil
```

Other predicates are useful too:

```elisp
(file-readable-p "~/.emacs.d/init.el")
(file-writable-p "~/.emacs.d/init.el")
(file-directory-p "~/.emacs.d")
(file-regular-p "~/.emacs.d/init.el")
```

Use these before doing file operations when overwriting, permissions, or file type matter.

## Create Directories

Use `make-directory`:

```elisp
(make-directory "~/.local/share/example")
```

If the directory already exists, this raises an error.

Pass `t` as the second argument to create missing parents and avoid complaining if the directory already exists:

```elisp
(make-directory "~/.local/share/example" t)
```

This is like `mkdir -p`.

## Ensure Output Directories

Before creating symlinks, the package should create common shared directories such as:

```text
~/.config
~/.local/share
```

Why?

If `~/.config` does not exist and the package creates it as a symlink to the dotfiles repo, then later application-generated config could accidentally appear inside the repository. That is annoying and noisy.

It is better to create `~/.config` and `~/.local/share` as real directories, then link individual application directories or files underneath them.

```elisp
(defun dotfiles--ensure-output-directories ()
  "Create directories listed in `dotfiles-ensure-output-directories'."
  (dolist (directory dotfiles-ensure-output-directories)
    (make-directory
     (expand-file-name directory dotfiles-output-directory)
     t)))
```

This keeps the later symlink algorithm simpler.

## List Files in a Directory

Use `directory-files` to list entries in one directory:

```elisp
(directory-files "~/.dotfiles")
```

Pass `t` as the second argument to return absolute paths:

```elisp
(directory-files "~/.dotfiles" t)
```

Pass a regular expression as the third argument to filter:

```elisp
(directory-files "~/.dotfiles" t "\\.org\\'")
```

That returns Org files.

There are more optional arguments, including sorting and result count limits, but these three are the usual ones:

```elisp
(directory-files DIRECTORY FULL MATCH)
```

## List Files Recursively

Use `directory-files-recursively` to walk a directory tree:

```elisp
(directory-files-recursively "~/.dotfiles" "\\.el\\'")
```

The second argument is required. It is a regular expression matching files to include.

Use an empty string to include all files:

```elisp
(directory-files-recursively "~/.dotfiles/files" "")
```

You can include directories in the result:

```elisp
(directory-files-recursively "~/.dotfiles/files" "" t)
```

You can also restrict which directories are traversed with a predicate:

```elisp
(directory-files-recursively
 "~/.emacs.d"
 "\\.el\\'"
 nil
 (lambda (directory)
   (string= (file-name-nondirectory directory) "lisp")))
```

And you can choose whether symbolic links should be followed. Check the function documentation with:

```text
C-h f directory-files-recursively
```

The exact argument list is worth reading before using the advanced options.

## Find Configuration Files to Link

The package can find every file under its config files directory:

```elisp
(defun dotfiles--config-files ()
  "Return all files under `dotfiles-config-files-directory'."
  (directory-files-recursively
   (dotfiles--config-files-path)
   ""))
```

To see where each file would go:

```elisp
(dolist (config-file (dotfiles--config-files))
  (message "%s -> %s"
           config-file
           (dotfiles--config-file-target config-file)))
```

At this point we can compute the source and target paths for every configuration file.

## Copy Files and Directories

Use `copy-file`:

```elisp
(copy-file "~/.emacs.d/init.el" "/tmp/")
```

The trailing slash matters. With `/tmp/`, Emacs copies the file into the directory. Without the slash, Emacs may treat `/tmp` as the destination file name.

If the destination exists, `copy-file` raises an error unless you pass a non-nil overwrite argument:

```elisp
(copy-file "~/.emacs.d/init.el" "/tmp/" t)
```

Use `copy-directory` for directories:

```elisp
(copy-directory "~/.emacs.d/lisp" "/tmp/")
```

`copy-directory` has optional arguments for copying contents, preserving metadata, and related behavior. Use `C-h f copy-directory` before depending on the less common options.

## Move or Rename Files

Use `rename-file`:

```elisp
(rename-file "/tmp/init.el" "/tmp/init-old.el")
```

The same function can move a file:

```elisp
(rename-file "/tmp/init.el" "~/.dotfiles/files/.emacs.d/init.el")
```

Use the optional overwrite argument if you intentionally want to replace an existing file:

```elisp
(rename-file "/tmp/init.el" "~/.dotfiles/files/.emacs.d/init.el" t)
```

Be careful with overwrite behavior in user-facing code. It is often better to detect conflicts and ask the user what to do.

## Delete Files and Directories

Use `delete-file` for files:

```elisp
(delete-file "/tmp/init.el")
```

Use `delete-directory` for directories:

```elisp
(delete-directory "/tmp/example")
```

If the directory is not empty, pass `t` to delete recursively:

```elisp
(delete-directory "/tmp/example" t)
```

Recursive deletion is powerful. Put guardrails around it in real package code.

## Move Existing Config into the Dotfiles Repo

It is useful to migrate an existing file from the output directory into the dotfiles repo.

For example:

```text
~/.config/geeks
```

should move to:

```text
~/.dotfiles/files/.config/geeks
```

The function should:

1. Prompt for a source path.
2. Verify that the source is inside the output directory.
3. Compute the corresponding destination path inside the dotfiles repo.
4. Refuse to overwrite an existing destination.
5. Create missing parent directories.
6. Move the file or directory.

```elisp
(defun dotfiles-move-to-config-files (source-path)
  "Move SOURCE-PATH into `dotfiles-config-files-directory'."
  (interactive "FConfiguration path to move: ")
  (let* ((relative-path
          (file-relative-name source-path dotfiles-output-directory))
         (destination-path
          (expand-file-name relative-path
                            (dotfiles--config-files-path)))
         (destination-path
          (if (string-suffix-p "/" destination-path)
              (substring destination-path 0 -1)
            destination-path)))
    (when (string-prefix-p ".." relative-path)
      (error "Path is not inside `dotfiles-output-directory': %s"
             source-path))
    (when (file-exists-p destination-path)
      (error "Destination already exists: %s" destination-path))
    (make-directory (file-name-directory destination-path) t)
    (rename-file source-path destination-path)))
```

This command moves files in the reverse direction of the linker:

```text
home directory -> dotfiles repo
```

The symlink code later makes them appear back in the home directory.

## Create Symbolic Links

Use `make-symbolic-link`:

```elisp
(make-symbolic-link
 "~/.dotfiles/files/.config/geeks"
 "~/.config/geeks")
```

The first argument is the link target: the real file or directory.

The second argument is the link name: the path where the symlink should be created.

Pass a non-nil third argument to avoid an error if the link name already exists:

```elisp
(make-symbolic-link
 "~/.dotfiles/files/.config/geeks"
 "~/.config/geeks"
 t)
```

On Windows, symbolic link creation can require administrator privileges or developer-mode support. Cross-platform dotfile tools may need a different strategy there.

## Inspect Symbolic Links

Use `file-symlink-p`:

```elisp
(file-symlink-p "~/.emacs.d")
```

If the path is a symbolic link, this returns its target. Otherwise it returns `nil`.

Use `file-truename` to resolve links in a path:

```elisp
(file-truename "~/.emacs.d/init.el")
```

Even if `init.el` itself is not a symlink, `file-truename` resolves any symlinks in the path leading to it.

## Link Configuration Files

Now the package can create symlinks for every config file.

The simple approach would create one symlink per file.

But a better approach is to link at the highest safe level. If the output directory already has:

```text
~/.local/share
```

and the dotfiles repo contains:

```text
~/.dotfiles/files/.local/share/applications/example.desktop
```

then the package can create one symlink:

```text
~/.local/share/applications -> ~/.dotfiles/files/.local/share/applications
```

instead of linking every file under `applications`.

The algorithm is:

1. Walk every file under the dotfiles config files directory.
2. Break each relative path into path parts.
3. Check each progressively deeper target path.
4. If the target is already the right symlink, stop.
5. If the target is a symlink to somewhere else, error.
6. If the target is an existing directory, keep walking deeper.
7. If the target does not exist, create a symlink there and stop.

```elisp
(defun dotfiles-link-config-files ()
  "Link all files in `dotfiles-config-files-directory'."
  (interactive)
  (dotfiles--ensure-output-directories)
  (dolist (config-file (dotfiles--config-files))
    (dotfiles--link-config-file config-file)))

(defun dotfiles--link-config-file (config-file)
  "Create an appropriate symbolic link for CONFIG-FILE."
  (let ((path-parts
         (split-string
          (file-relative-name config-file
                              (dotfiles--config-files-path))
          "/"
          t))
        (current-path nil))
    (while path-parts
      (setq current-path
            (if current-path
                (concat current-path "/" (car path-parts))
              (car path-parts)))
      (setq path-parts (cdr path-parts))
      (let ((source-path
             (expand-file-name current-path
                               (dotfiles--config-files-path)))
            (target-path
             (expand-file-name current-path
                               dotfiles-output-directory)))
        (cond
         ((file-symlink-p target-path)
          (if (string= (file-truename target-path)
                       (file-truename source-path))
              (setq path-parts nil)
            (error "Target is a symlink to another file: %s"
                   target-path)))
         ((file-directory-p target-path)
          ;; Keep walking into the existing directory.
          nil)
         (t
          (make-symbolic-link source-path target-path)
          (setq path-parts nil)))))))
```

This reuses the same idea as GNU Stow: link as high in the tree as possible, but do not replace real directories that should contain files from multiple applications.

There is room for more safety here. For example, if `target-path` exists as a regular file, `make-symbolic-link` will signal an error. A production package might catch that error and present a clearer recovery path to the user.

## Put It All Together

Now the package can expose one command that does the full update:

```elisp
(defun dotfiles-update ()
  "Tangle Org files, link config files, and update `.gitignore'."
  (interactive)
  (dotfiles-tangle-org-files)
  (dotfiles-link-config-files)
  (dotfiles--update-gitignore)
  (message "Dotfiles are up to date"))
```

This ties together the work from the last several lessons:

- `dotfiles-tangle-org-files` generates files from Org configuration.
- `dotfiles-link-config-files` creates symlinks for ordinary config files.
- `dotfiles--update-gitignore` ignores generated tangled files.

The command should be repeatable. If everything is already in the right state, running it again should leave things as they are.

That is a good property for configuration management code. You want "make my dotfiles current" to be something you can run after pulling changes, switching machines, or editing Org files.

## What to Remember

The key ideas:

- `default-directory` is the current directory for a buffer.
- `expand-file-name` resolves relative paths.
- `file-relative-name` computes one path relative to another.
- `file-name-directory`, `file-name-nondirectory`, `file-name-extension`, and related helpers split paths apart.
- `file-exists-p` and friends test filesystem state.
- `make-directory` creates directories, and its second argument behaves like `mkdir -p`.
- `directory-files` lists one directory.
- `directory-files-recursively` walks a directory tree.
- `copy-file`, `copy-directory`, `rename-file`, `delete-file`, and `delete-directory` cover common file operations.
- `make-symbolic-link` creates symlinks.
- `file-symlink-p` detects symlinks.
- `file-truename` resolves symlinked paths.

Once you can calculate paths reliably, most file automation becomes a sequence of small checks and transformations. That is the quiet magic here: the filesystem stops being something you poke manually and becomes another part of Emacs you can program.
