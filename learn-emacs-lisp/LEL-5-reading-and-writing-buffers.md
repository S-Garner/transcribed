# Learn Emacs Lisp 5: Reading and Writing Buffers

Buffers are where most Emacs Lisp work becomes real.

You can evaluate expressions, define functions, and set variables without touching a buffer directly. But the moment you want to automate editing, generate files, create custom displays, communicate with external programs, or build a package that feels native to Emacs, you need to understand buffers.

This tutorial covers:

- what buffers are
- the current buffer
- finding or creating buffers
- visiting files without displaying them
- point and movement
- reading text from buffers
- searching for text
- inserting and deleting text
- saving buffers
- using these pieces to update a generated `.gitignore`

The practical example continues the dotfiles helper from earlier episodes. The goal is to scan Org configuration files for `:tangle` output paths and automatically add those generated files to `.gitignore`.

## What Is a Buffer?

A buffer is an Emacs object that contains text.

That text may come from a file:

```text
~/.emacs.d/init.el
```

or it may be created entirely by Emacs:

```text
*scratch*
*Messages*
*Help*
```

A buffer does not have to be visible. Emacs Lisp code can create a buffer, insert text into it, read from it, save it to disk, or throw it away without ever showing it in a window.

This matters because many Emacs features are built on buffers:

- editing files
- help pages
- compilation output
- Magit status views
- REPLs
- shell sessions
- package UIs
- generated reports

Buffers are not just text boxes. They can also have buffer-local variables, modes, point positions, text properties, and process connections.

## The Current Buffer

Many buffer functions operate on the current buffer.

You can inspect it with:

```elisp
(current-buffer)
```

This returns the buffer object Emacs Lisp is currently operating on.

That is usually the buffer visible in the selected window, but not always. Code can temporarily make another buffer current while it does work.

## Get a Buffer by Name

Use `get-buffer` to look up an existing buffer:

```elisp
(get-buffer "*scratch*")
```

If the buffer exists, Emacs returns it.

If it does not exist, Emacs returns `nil`.

Use `get-buffer-create` when you want to create the buffer if needed:

```elisp
(get-buffer-create "hello system crafters")
```

This creates a new empty buffer named `hello system crafters` if one does not already exist.

It does not display the buffer. It only returns the buffer object.

## Temporarily Work in Another Buffer

You can change the current buffer directly:

```elisp
(set-buffer "*scratch*")
```

or:

```elisp
(set-buffer (get-buffer "*scratch*"))
```

This works, but it is easy to misuse. If code changes the current buffer and does not restore it, later code may unexpectedly operate on the wrong buffer.

Use `save-current-buffer` when you need to change buffers and then restore the original:

```elisp
(save-current-buffer
  (set-buffer "*scratch*")
  (message "Current buffer: %s" (current-buffer)))
```

After the body finishes, Emacs restores the previous current buffer.

Most of the time, `with-current-buffer` is clearer:

```elisp
(with-current-buffer "*scratch*"
  (message "Current buffer: %s" (current-buffer)))
```

`with-current-buffer` makes a buffer current only while evaluating its body.

## File-Backed Buffers

Some buffers are visiting files.

In a file-backed buffer, this returns the absolute path:

```elisp
(buffer-file-name)
```

In a non-file buffer like `*scratch*`, it returns `nil`.

To get the buffer visiting a specific file, use `get-file-buffer`:

```elisp
(get-file-buffer "/home/user/.dotfiles/emacs.org")
```

If that file is already open, Emacs returns its buffer. If not, it returns `nil`.

## `default-directory`

Relative paths are resolved against `default-directory`.

You can inspect it with:

```text
C-h v default-directory
```

`default-directory` is usually buffer-local. In a file buffer, it is normally the directory containing that file.

That means this:

```elisp
(expand-file-name "emacs.org" dotfiles-folder)
```

is safer than relying on whatever `default-directory` happens to be.

For the dotfiles helper, assume these customization variables from earlier episodes:

```elisp
(defcustom dotfiles-folder "~/.dotfiles"
  "The path to the dotfiles folder."
  :type 'string
  :group 'dotfiles)

(defcustom dotfiles-org-files '("emacs.org" "desktop.org" "systems.org")
  "List of Org files to tangle in `dotfiles-folder'."
  :type '(repeat string)
  :group 'dotfiles)
```

You can loop over those Org files and look up already-open buffers:

```elisp
(dolist (org-file dotfiles-org-files)
  (with-current-buffer
      (get-file-buffer
       (expand-file-name org-file dotfiles-folder))
    (message "File: %s" (buffer-file-name))))
```

That only works if the files are already open.

## Visit a File Without Displaying It

Use `find-file-noselect` to visit a file in a buffer without selecting or displaying it:

```elisp
(find-file-noselect "~/.dotfiles/emacs.org")
```

This is useful for package code. You often need to operate on a file, but you do not want to steal the user's window focus.

A common pattern is:

```elisp
(let ((file-path (expand-file-name org-file dotfiles-folder)))
  (with-current-buffer
      (or (get-file-buffer file-path)
          (find-file-noselect file-path))
    (message "File: %s" (buffer-file-name))))
```

The `or` expression tries `get-file-buffer` first.

If the file is already open, use that buffer. If not, visit it with `find-file-noselect`.

This is a good habit because the open buffer may contain unsaved user edits that are newer than the file on disk.

## Point

Point is the current editing position in a buffer.

You can inspect it with:

```elisp
(point)
```

Point is an integer. The first accessible character position in a normal buffer is usually `1`.

Use these to get the accessible bounds of the buffer:

```elisp
(point-min)
(point-max)
```

They usually mean "beginning of buffer" and "end of buffer", but narrowing can change what part of the buffer is accessible. That is why these functions are better than hard-coding `1` or assuming the physical end of the buffer.

## Move Point

Move point to an exact location:

```elisp
(goto-char (point-min))
(goto-char (point-max))
```

Move by characters:

```elisp
(forward-char)
(forward-char 5)
(backward-char)
(backward-char 5)
```

Move by words:

```elisp
(forward-word)
(backward-word)
```

Many familiar Emacs keybindings call movement functions like these. `C-f`, `C-b`, `C-n`, and `C-p` are not magical editor primitives. They are commands that move point.

## Preserve the User's Point

When package code moves point in a user's buffer, restore it afterward.

Use `save-excursion`:

```elisp
(save-excursion
  (goto-char (point-min))
  ;; Do buffer work here.
  )
```

This lets your code search, insert, or delete text without leaving the user's cursor somewhere surprising.

When writing buffer-manipulation code, `with-current-buffer` and `save-excursion` often appear together:

```elisp
(with-current-buffer some-buffer
  (save-excursion
    (goto-char (point-min))
    ;; Work in SOME-BUFFER without permanently moving point.
    ))
```

## Read Characters and Strings

Use `char-after` to inspect the character after point:

```elisp
(char-after)
```

You can also pass a position:

```elisp
(char-after (point-min))
```

Use `buffer-substring` to copy text from a region:

```elisp
(buffer-substring (point-min) (point-max))
```

This preserves text properties. In rich buffers like Org buffers, that may include display or face information.

If you want plain text, use:

```elisp
(buffer-substring-no-properties (point-min) (point-max))
```

For most file-generation and parsing code, the no-properties version is the one you want.

## Read Things at Point

The `thing-at-point` function extracts a meaningful object near point:

```elisp
(thing-at-point 'word)
```

The first argument says what kind of thing to read.

Examples:

```elisp
(thing-at-point 'word)
(thing-at-point 'sentence)
(thing-at-point 'sexp)
(thing-at-point 'filename)
```

Pass `t` as the second argument to remove text properties:

```elisp
(thing-at-point 'filename t)
```

This is very useful for code that scans buffers. In this episode's example, after searching for `:tangle `, point will be sitting just before the output path. `thing-at-point` can then grab that file name.

## Search for Text

Use `search-forward` to search from point toward the end of the buffer:

```elisp
(search-forward ":tangle ")
```

After a successful match, point moves to the end of the matched text.

Use `search-backward` to search toward the beginning:

```elisp
(search-backward ":tangle ")
```

After a successful backward match, point moves to the beginning of the matched text.

By default, these functions signal an error if they do not find a match.

For programmatic searches, pass `t` for the `NOERROR` argument:

```elisp
(search-forward ":tangle " nil t)
```

The second argument is the search bound. `nil` means no explicit bound.

With `NOERROR` set to `t`, the function returns `nil` instead of raising an error when no match is found.

There is also a count argument:

```elisp
(search-forward ":tangle " nil t 3)
```

That finds the third match after point.

Regular-expression search functions such as `re-search-forward` are more powerful, but plain string search is enough for this example.

## Example: Find Org Code Block Output Paths

Org Babel source blocks can use `:tangle` to say where generated code should be written:

```org
#+begin_src emacs-lisp :tangle .emacs.d/init.el
(message "hello")
#+end_src
```

The simple approach in this episode is:

1. Open each Org file.
2. Search for `:tangle `.
3. Read the filename after it.
4. Collect those filenames into a list.

This does not cover every Org feature. Org properties can be inherited, and real-world Babel header arguments can be more complex. But this is a useful first pass and a good buffer API exercise.

```elisp
(defun dotfiles--scan-for-output-files (org-file)
  "Return a list of tangled output files found in ORG-FILE."
  (let ((output-files '())
        (current-match t))
    (with-current-buffer
        (or (get-file-buffer org-file)
            (find-file-noselect org-file))
      (save-excursion
        (goto-char (point-min))
        (while current-match
          (setq current-match
                (search-forward ":tangle " nil t))
          (when current-match
            (let ((output-file (thing-at-point 'filename t)))
              (unless (or (not output-file)
                          (string= output-file "no"))
                (setq output-files
                      (cons output-file output-files))))))))
    output-files))
```

The double hyphen in `dotfiles--scan-for-output-files` marks it as an internal helper. It is not meant to be called directly by users.

Test it across all configured Org files:

```elisp
(let ((output-files '()))
  (dolist (org-file dotfiles-org-files)
    (setq output-files
          (append output-files
                  (dotfiles--scan-for-output-files
                   (expand-file-name org-file dotfiles-folder)))))
  output-files)
```

This produces one list of generated output paths.

## Insert Text

Use `insert` to add text at point:

```elisp
(insert "hello")
```

`insert` can take several arguments:

```elisp
(insert "\n"
        "This is Sparta"
        ?!
        ?\n)
```

Characters use the `?` syntax:

```elisp
?!
?\n
```

Use `insert-char` when you want to insert one character repeatedly:

```elisp
(insert-char ?- 20)
```

That inserts 20 hyphens.

## Example: Write Generated Files into `.gitignore`

The dotfiles package should let the user manage part of `.gitignore` manually while the package manages the generated section.

Define a marker string:

```elisp
(defvar dotfiles--gitignore-marker
  "# Generated by dotfiles.el")
```

Everything before this marker belongs to the user. Everything after it can be regenerated by the package.

A first version of the updater might look like this:

```elisp
(defun dotfiles--update-gitignore ()
  "Update `.gitignore' with generated dotfiles output paths."
  (let ((output-files '()))
    (dolist (org-file dotfiles-org-files)
      (setq output-files
            (append output-files
                    (dotfiles--scan-for-output-files
                     (expand-file-name org-file dotfiles-folder)))))
    (let ((gitignore-file (expand-file-name ".gitignore" dotfiles-folder)))
      (with-current-buffer
          (or (get-file-buffer gitignore-file)
              (find-file-noselect gitignore-file))
        (save-excursion
          (goto-char (point-min))
          (if (search-forward dotfiles--gitignore-marker nil t)
              (progn
                (end-of-line)
                (forward-line 1))
            (goto-char (point-max))
            (unless (bolp)
              (insert "\n"))
            (insert dotfiles--gitignore-marker "\n"))
          (dolist (output-file output-files)
            (insert output-file "\n")))))))
```

This finds or creates the generated section and appends output paths.

There is a problem: every time the function runs, it appends the same paths again.

We need to delete the old generated section before writing the new one.

## Delete Text

Use `delete-region` to remove text between two positions:

```elisp
(delete-region START END)
```

For example, this deletes from point to the end of the accessible buffer:

```elisp
(delete-region (point) (point-max))
```

This is exactly what the `.gitignore` updater needs after it finds the generated marker.

## Save a Buffer

Use `save-buffer` to save the current buffer to its visited file:

```elisp
(save-buffer)
```

If the buffer is visiting `.gitignore`, this writes the updated ignore file to disk.

If the buffer is not visiting a file, Emacs may prompt for a file name.

## Example: Clean Up and Save `.gitignore`

Here is the corrected updater:

```elisp
(defun dotfiles--update-gitignore ()
  "Update `.gitignore' with generated dotfiles output paths."
  (let ((output-files '()))
    (dolist (org-file dotfiles-org-files)
      (setq output-files
            (append output-files
                    (dotfiles--scan-for-output-files
                     (expand-file-name org-file dotfiles-folder)))))
    (let ((gitignore-file (expand-file-name ".gitignore" dotfiles-folder)))
      (with-current-buffer
          (or (get-file-buffer gitignore-file)
              (find-file-noselect gitignore-file))
        (save-excursion
          (goto-char (point-min))
          (if (search-forward dotfiles--gitignore-marker nil t)
              (progn
                (end-of-line)
                (forward-line 1))
            (goto-char (point-max))
            (unless (bolp)
              (insert "\n"))
            (insert dotfiles--gitignore-marker "\n"))
          (delete-region (point) (point-max))
          (dolist (output-file output-files)
            (insert output-file "\n"))
          (save-buffer))))))
```

Now the function is repeatable:

1. Find or create the generated marker.
2. Delete everything after it.
3. Insert the current list of generated files.
4. Save the buffer.

Running it twice should leave the file with the same number of lines, assuming the tangled output list has not changed.

## What to Remember

The key ideas:

- Buffers are the main text containers in Emacs.
- A buffer may or may not be visiting a file.
- Many functions operate on the current buffer.
- `with-current-buffer` temporarily changes the current buffer.
- `get-buffer-create` creates non-file buffers.
- `get-file-buffer` finds an already-open file buffer.
- `find-file-noselect` visits a file without displaying it.
- Point is the current editing position.
- `save-excursion` restores point after buffer operations.
- `buffer-substring-no-properties` reads plain text from a buffer.
- `thing-at-point` extracts useful text near point.
- `search-forward` and `search-backward` move point while searching.
- `insert` writes text at point.
- `delete-region` removes text between positions.
- `save-buffer` writes a file-backed buffer to disk.

These functions are enough to build useful automation. Once you can move around a buffer, read text, search for markers, insert generated content, delete stale content, and save the result, Emacs becomes a programmable text workshop rather than just an editor.
