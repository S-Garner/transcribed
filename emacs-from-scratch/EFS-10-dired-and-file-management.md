# Emacs From Scratch 10: Dired and File Management

Dired is Emacs' built-in file manager. It looks plain at first because a Dired buffer is mostly a directory listing, but that simplicity is the point: once files are shown as text, Emacs can search them, mark them, edit them, rename them, copy them, delete them, and batch-process them with the same habits you already use elsewhere.

This tutorial covers the core Dired workflow and a few packages that make it feel smoother for day-to-day file management.

## Open Dired

Start Dired with:

```text
M-x dired
```

or with the default key binding:

```text
C-x d
```

Emacs asks for a directory and opens a Dired buffer for it. On Linux and macOS, Dired uses the system `ls` command to produce the listing. On Windows, Emacs emulates enough of that behavior internally.

The buffer may look like a normal `ls -l` listing, but each line is interactive. You can move through files, open them, mark them for later commands, rename them, copy them, and delete them.

## Navigate Files

Useful default navigation commands:

```text
n       next line
p       previous line
RET     open file or directory
^       go to parent directory
g       refresh the Dired buffer
```

If you use Evil Collection, Dired also gets more Vim-like movement:

```text
j       next line
k       previous line
```

When you are unsure what a Dired buffer can do, ask Emacs:

```text
C-h m   describe the current mode
C-h b   show active key bindings
```

If you use `which-key`, you can also inspect a keymap directly:

```text
M-x which-key-show-keymap
```

Then choose `dired-mode-map`.

## Open Files in Other Ways

`RET` opens the file or directory under point. Dired also gives you several related commands that are useful when browsing.

Open in another window:

```text
S-RET
```

Display the file in another window without selecting that window:

```text
M-RET
```

View a file temporarily:

```text
g o
```

That runs `dired-view-file`. It is useful when you want to glance at a file and then press `q` to return.

## Jump From a File to Dired

`dired-jump` opens Dired in the directory of the current file and places point on that file.

This is a good binding to add:

```elisp
(use-package dired
  :ensure nil
  :commands (dired dired-jump)
  :bind (("C-x C-j" . dired-jump)))
```

Now, while editing a file, press:

```text
C-x C-j
```

You will land in Dired at that file's location.

## Configure the Listing

Dired's listing is controlled by `dired-listing-switches`.

For a cleaner listing on GNU/Linux:

```elisp
(use-package dired
  :ensure nil
  :commands (dired dired-jump)
  :bind (("C-x C-j" . dired-jump))
  :custom
  (dired-listing-switches "-agho --group-directories-first"))
```

This keeps hidden files, hides some owner/group noise, uses human-readable sizes, and puts directories first.

The `--group-directories-first` option is specific to GNU `ls`. If you are on macOS, the default BSD `ls` may not support it unless you install GNU coreutils and configure Emacs to use that version.

You can also hide or show the detailed metadata columns interactively:

```text
(
```

That toggles `dired-hide-details-mode`.

## Mark Files

Dired's real strength appears when you mark files and then run one command across all marked files.

Common marking commands:

```text
m       mark file
u       unmark file
U       unmark all
t       toggle marked and unmarked files
```

Mark files by regular expression:

```text
% m
```

Mark files by extension:

```text
* .
```

Some of the extra `*` commands come from `dired-x`, which ships with Emacs:

```elisp
(require 'dired-x)
```

Other useful marking commands include marking directories, executables, symbolic links, and similar file classes. Use `C-h b` in Dired to see the bindings available in your setup.

If you want to hide marked lines from the current Dired view without deleting the files, use:

```text
k
```

Refresh the buffer to bring them back:

```text
g
```

With Evil Collection, refresh is commonly available as:

```text
g r
```

## Copy, Move, and Rename

Copy the file under point, or all marked files:

```text
C
```

Rename or move the file under point, or all marked files:

```text
R
```

When multiple files are marked, the target should usually be a directory.

If you use Ivy and you want to rename a file to a new name that does not already exist, Ivy may try to complete to an existing candidate. Use:

```text
C-M-j
```

That runs `ivy-immediate-done`, accepting exactly what you typed.

## Use Two Dired Windows

Dired can guess that the other visible Dired buffer is your target.

Enable this with:

```elisp
(setq dired-dwim-target t)
```

Now, if you have two Dired buffers open side by side, copy and rename operations will default to the directory shown in the other window. This makes two-pane file management feel much more natural.

## Delete Files Safely

Delete immediately:

```text
D
```

Mark for deletion:

```text
d
```

Execute marked deletions:

```text
x
```

If you want deletions to go to the system trash instead of being removed permanently, enable:

```elisp
(setq delete-by-moving-to-trash t)
```

This is a good default if you use Dired for real file management.

## Compress and Extract Files

Compress or uncompress using Dired's default behavior:

```text
Z
```

On normal files, this creates an archive such as a `.tar.gz`. On an archive, it extracts it.

Choose an archive name explicitly:

```text
c
```

The archive extension controls the command Dired uses. For example, a `.zip` name will use zip support if available.

The mapping is controlled by:

```elisp
dired-compress-files-alist
```

You can customize that variable if you want to add tools such as `7z`.

## Change File Attributes

Dired can run common file maintenance operations without leaving Emacs.

Touch or change timestamps:

```text
T
```

Change permissions:

```text
M
```

Change owner:

```text
O
```

Change group:

```text
G
```

Create symbolic links:

```text
S
```

Some bindings may differ if Evil Collection has claimed a key, so use `C-h b` if a command does not appear where you expect.

## Edit Filenames Directly

One of Dired's best features is writable Dired.

Enter writable mode:

```text
C-x C-q
```

Now the filenames in the Dired buffer are editable text. You can use normal Emacs editing commands, search and replace, keyboard macros, or Evil substitutions to rename files in bulk.

After editing, commit the changes:

```text
C-c C-c
```

Abort the edit:

```text
C-c C-k
```

With Evil, you may also be able to use:

```text
ZZ     commit
ZQ     abort
```

Writable Dired is often easier than memorizing every specialized batch command because you can use the text-editing tools you already know.

## Rename with Regular Expressions

Dired also supports regex-based batch renaming.

Mark some files, then run:

```text
% R
```

You will be prompted for a regular expression and a replacement.

For example, to prefix every marked file whose name begins with `habit`:

```text
Regexp:      ^habit
Replacement: my&
```

In the replacement, `&` means "the text matched by the whole regexp." So `habit.org` becomes `myhabit.org`.

For many renaming jobs, though, writable Dired is easier. Press `C-x C-q`, use normal search and replace across the filenames, then commit the changes.

## Reuse One Dired Buffer

By default, Dired creates many buffers as you move through directories. That can be useful, but it can also clutter your buffer list.

The `dired-single` package changes directory navigation so that Dired reuses the same buffer.

```elisp
(use-package dired-single)
```

If you use Evil Collection, you can make `h` and `l` behave like a file manager:

```elisp
(use-package dired
  :ensure nil
  :config
  (evil-collection-define-key 'normal 'dired-mode-map
    "h" 'dired-single-up-directory
    "l" 'dired-single-buffer))
```

This makes `h` go up a directory and `l` open the selected file or directory in the same Dired buffer.

The tradeoff is that some workflows benefit from keeping separate Dired buffers around, especially two-pane copy and move workflows. If `dired-single` feels too aggressive, use standard Dired navigation instead.

## Add Icons

If you already use `all-the-icons`, you can add icons to Dired with `all-the-icons-dired`.

```elisp
(use-package all-the-icons-dired
  :hook (dired-mode . all-the-icons-dired-mode))
```

This is mostly visual, but it can make files easier to scan quickly. It requires the icon fonts to be installed.

## Open Files Externally

Dired can run external commands on files.

Run a command asynchronously:

```text
&
```

Run a command synchronously:

```text
!
```

The synchronous version blocks Emacs until the command exits, so `&` is usually better for opening media files or GUI programs.

For extension-based opening, use `dired-open`:

```elisp
(use-package dired-open
  :config
  (setq dired-open-extensions
        '(("png" . "feh")
          ("jpg" . "feh")
          ("mkv" . "mpv")
          ("mp4" . "mpv"))))
```

Now Dired can open those file types with the programs you prefer.

There is also `dired-open-xdg`, which can use desktop file associations on Linux. Explicit extension mappings are often more predictable, especially if you do not want directories to open in an external file manager.

## Hide Dotfiles

If you want Dired to hide dotfiles by default, use `dired-hide-dotfiles`.

```elisp
(use-package dired-hide-dotfiles
  :hook (dired-mode . dired-hide-dotfiles-mode)
  :config
  (evil-collection-define-key 'normal 'dired-mode-map
    "H" 'dired-hide-dotfiles-mode))
```

Now dotfiles are hidden when Dired opens, and `H` toggles them.

Another built-in option is `dired-omit-mode`, which is more general. `dired-hide-dotfiles` is simpler if all you want is a quick dotfile toggle.

## Insert Subdirectories Inline

Dired can show a subdirectory inside the current Dired buffer instead of opening another buffer.

Run:

```text
i
```

That calls `dired-maybe-insert-subdir`. In some Evil setups, the binding may be `I`.

This can be useful when you want to inspect several related directories in one buffer and then operate on files across them.

## A Practical Dired Configuration

Here is a compact configuration that pulls the main ideas together:

```elisp
(use-package dired
  :ensure nil
  :commands (dired dired-jump)
  :bind (("C-x C-j" . dired-jump))
  :custom
  ((dired-listing-switches "-agho --group-directories-first")
   (dired-dwim-target t)
   (delete-by-moving-to-trash t)))

(require 'dired-x)

(use-package dired-single)

(use-package all-the-icons-dired
  :hook (dired-mode . all-the-icons-dired-mode))

(use-package dired-open
  :config
  (setq dired-open-extensions
        '(("png" . "feh")
          ("jpg" . "feh")
          ("mkv" . "mpv")
          ("mp4" . "mpv"))))

(use-package dired-hide-dotfiles
  :hook (dired-mode . dired-hide-dotfiles-mode))
```

If you use Evil Collection and `dired-single`, add:

```elisp
(evil-collection-define-key 'normal 'dired-mode-map
  "h" 'dired-single-up-directory
  "l" 'dired-single-buffer
  "H" 'dired-hide-dotfiles-mode)
```

## When to Use Dired

Dired is worth learning because it is not only a file browser. It is a file-management buffer.

Use it when you want to:

- browse directories without leaving Emacs
- copy or move groups of files
- batch rename files
- edit filenames with normal text editing commands
- compress or extract archives
- open files in external applications
- manage files over remote connections with TRAMP

The first few minutes can feel strange because Dired does not look like a modern graphical file manager. After that, the trick becomes clear: your file manager is just another Emacs buffer, and that makes it programmable, searchable, editable, and composable with everything else.
