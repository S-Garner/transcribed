# Learn Emacs Lisp 7: Custom Minor Modes

Modes are one of the main ways Emacs packages organize behavior.

Major modes define the main editing experience for a buffer. Minor modes layer extra behavior on top. If you want a package to provide keybindings, mode-line state, hooks, and optional behavior that can be turned on and off, a minor mode is often the right shape.

This tutorial covers:

- major modes vs minor modes
- what a minor mode needs internally
- building a minor mode by hand
- creating hooks
- using `define-minor-mode`
- adding a mode keymap
- creating a global minor mode for the dotfiles package
- automatically tangling Org config files on save

The practical example continues the dotfiles package from earlier lessons. The goal is to create a minor mode that tangles known Org configuration files when they are saved and updates linked configuration files afterward.

## What Is a Mode?

Most user-facing Emacs behavior comes from modes.

A major mode is the primary mode for a buffer.

Examples:

- `org-mode`
- `emacs-lisp-mode`
- `python-mode`
- `dired-mode`

A buffer can have only one major mode at a time. If you switch from `org-mode` to `emacs-lisp-mode`, Emacs does not add a second major mode. It replaces the current one.

A minor mode adds supporting behavior.

Examples:

- `auto-fill-mode`
- `display-line-numbers-mode`
- `flyspell-mode`
- `global-auto-revert-mode`

A buffer can have many minor modes enabled at the same time. Some minor modes are buffer-local. Others are global and affect Emacs more broadly.

## What Modes Can Provide

A mode can provide:

- keybindings that are active only while the mode is enabled
- a mode-line indicator
- setup or cleanup code
- hooks that other users or packages can attach to
- buffer-local or global state

For example, when `org-mode` starts, it enables Org-specific keybindings, syntax behavior, indentation rules, and hooks.

For a dotfiles package, a minor mode is useful because it can:

- expose package keybindings under one prefix
- enable automatic tangling on save
- let users turn the behavior on or off
- keep package bindings out of the global keymap unless the mode is active

## Mode Commands

Modes are controlled by commands.

You can run a mode interactively:

```text
M-x org-mode
M-x auto-fill-mode
M-x display-line-numbers-mode
```

You can also call mode functions from Emacs Lisp:

```elisp
(auto-fill-mode 1)
(auto-fill-mode -1)
```

By convention:

- a positive argument enables a minor mode
- a zero or negative argument disables it
- no interactive prefix usually toggles it
- calling from Lisp with no argument normally enables it

These conventions are a little odd at first, but Emacs tooling expects them.

## What a Minor Mode Needs

A hand-written minor mode usually has:

- a variable named after the mode, such as `dotfiles-basic-mode`
- a command with the same name
- an optional keymap
- an optional mode-line lighter
- optional hooks
- setup and cleanup code

The variable stores whether the mode is active. If it is `nil`, the mode is off. If it is non-nil, the mode is on.

If the mode is buffer-local, that variable should be buffer-local too.

## A Basic Minor Mode from Scratch

Here is a simple buffer-local minor mode written by hand:

```elisp
(defvar dotfiles-basic-mode nil
  "Non-nil if `dotfiles-basic-mode' is enabled.")

(make-variable-buffer-local 'dotfiles-basic-mode)

(defvar dotfiles-basic-mode-map
  (let ((map (make-sparse-keymap)))
    (define-key map (kbd "C-c C-. t")
                (lambda ()
                  (interactive)
                  (message "dotfiles keybinding used")))
    map)
  "Keymap for `dotfiles-basic-mode'.")

(add-to-list 'minor-mode-alist
             '(dotfiles-basic-mode " Dotfiles"))

(add-to-list 'minor-mode-map-alist
             (cons 'dotfiles-basic-mode
                   dotfiles-basic-mode-map))

(defun dotfiles-basic-mode (&optional arg)
  "Toggle `dotfiles-basic-mode'."
  (interactive (list 'toggle))
  (setq dotfiles-basic-mode
        (if (eq arg 'toggle)
            (not dotfiles-basic-mode)
          (> (prefix-numeric-value arg) 0)))
  (if dotfiles-basic-mode
      (message "dotfiles-basic-mode activated")
    (message "dotfiles-basic-mode deactivated")))
```

There are several pieces here.

The variable tracks state:

```elisp
(defvar dotfiles-basic-mode nil
  "Non-nil if `dotfiles-basic-mode' is enabled.")
```

The mode is buffer-local:

```elisp
(make-variable-buffer-local 'dotfiles-basic-mode)
```

The keymap is sparse, meaning it starts empty and only stores bindings you add:

```elisp
(make-sparse-keymap)
```

The mode-line lighter is registered in `minor-mode-alist`:

```elisp
(add-to-list 'minor-mode-alist
             '(dotfiles-basic-mode " Dotfiles"))
```

The leading space in `" Dotfiles"` is intentional. Minor mode lighters are usually displayed next to each other, so they include their own spacing.

The keymap is registered in `minor-mode-map-alist`:

```elisp
(add-to-list 'minor-mode-map-alist
             (cons 'dotfiles-basic-mode
                   dotfiles-basic-mode-map))
```

When `dotfiles-basic-mode` is non-nil, Emacs activates `dotfiles-basic-mode-map`.

## About the Interactive Argument

The mode command accepts an optional argument:

```elisp
(defun dotfiles-basic-mode (&optional arg)
  ...)
```

This line controls the interactive call:

```elisp
(interactive (list 'toggle))
```

When a user runs `M-x dotfiles-basic-mode`, the function receives the symbol `toggle`.

Then the mode decides what to do:

```elisp
(setq dotfiles-basic-mode
      (if (eq arg 'toggle)
          (not dotfiles-basic-mode)
        (> (prefix-numeric-value arg) 0)))
```

If `arg` is `toggle`, reverse the current value.

Otherwise, use the numeric prefix argument. Positive means on; zero or negative means off.

This is one of those tiny Emacs conventions that feels fussy until you let a macro do it for you.

## Hooks

A hook is a variable containing functions to run when something happens.

Define a hook variable:

```elisp
(defvar dotfiles-basic-mode-hook nil
  "Hook run after toggling `dotfiles-basic-mode'.")
```

Run it with:

```elisp
(run-hooks 'dotfiles-basic-mode-hook)
```

Add a function to it:

```elisp
(add-hook 'dotfiles-basic-mode-hook
          (lambda ()
            (message "dotfiles-basic-mode hook executed")))
```

Then update the mode command:

```elisp
(defun dotfiles-basic-mode (&optional arg)
  "Toggle `dotfiles-basic-mode'."
  (interactive (list 'toggle))
  (setq dotfiles-basic-mode
        (if (eq arg 'toggle)
            (not dotfiles-basic-mode)
          (> (prefix-numeric-value arg) 0)))
  (if dotfiles-basic-mode
      (message "dotfiles-basic-mode activated")
    (message "dotfiles-basic-mode deactivated"))
  (run-hooks 'dotfiles-basic-mode-hook))
```

For minor modes, use `run-hooks`.

For major modes, use `run-mode-hooks`. Major mode hooks have extra behavior for file-local variables, directory-local variables, and other mode initialization details.

## Use `define-minor-mode`

Writing a minor mode by hand is useful once. After that, use the macro.

`define-minor-mode` creates the mode variable, command, hooks, lighter, and keymap plumbing for you.

Here is the same kind of mode using `define-minor-mode`:

```elisp
(define-minor-mode dotfiles-mode
  "Toggle global Dotfiles mode."
  :init-value nil
  :global t
  :group 'dotfiles
  :lighter " Dotfiles"
  :keymap `((,(kbd "C-c C-. t")
             . ,(lambda ()
                  (interactive)
                  (message "dotfiles keybinding used"))))
  (if dotfiles-mode
      (message "dotfiles-mode activated")
    (message "dotfiles-mode deactivated")))
```

That is much less machinery.

The important keyword arguments are:

- `:init-value` sets the initial state.
- `:global t` makes this a global minor mode.
- `:group` associates the generated custom variable with a customization group.
- `:lighter` sets the mode-line indicator.
- `:keymap` defines mode-specific keybindings.

The body runs when the mode is enabled or disabled.

Inside the body, check the mode variable:

```elisp
(if dotfiles-mode
    ;; enabled
  ;; disabled
  )
```

## Generated Hooks

`define-minor-mode` creates a normal hook:

```elisp
dotfiles-mode-hook
```

It runs when the mode is toggled.

It also supports on/off hooks by convention:

```elisp
dotfiles-mode-on-hook
dotfiles-mode-off-hook
```

You can use them like this:

```elisp
(add-hook 'dotfiles-mode-on-hook
          (lambda ()
            (message "dotfiles-mode turned on")))

(add-hook 'dotfiles-mode-off-hook
          (lambda ()
            (message "dotfiles-mode turned off")))
```

Those hooks are useful when setup and cleanup should be separate.

## Dotfiles Mode Behavior

Now we can build the real mode.

When `dotfiles-mode` is enabled, it should:

- provide package keybindings
- watch Org buffers
- automatically tangle known Org config files on save
- update symlinks after tangling

It should not tangle every Org file.

It should only tangle files that:

- are listed in `dotfiles-org-files`
- live in `dotfiles-folder`
- are saved while `dotfiles-mode` is enabled
- are saved while `dotfiles-tangle-on-save` is non-nil

## Custom Key Prefix

Packages should avoid stealing arbitrary global keybindings.

Define a user-customizable prefix:

```elisp
(defcustom dotfiles-key-prefix (kbd "C-c C-.")
  "Prefix key for Dotfiles commands."
  :type 'key-sequence
  :group 'dotfiles)
```

Then create a small helper:

```elisp
(defun dotfiles--key (key)
  "Return KEY with `dotfiles-key-prefix' prepended."
  (vconcat dotfiles-key-prefix
           (kbd key)))
```

Now mode bindings can be written relative to the prefix:

```elisp
(dotfiles--key "t")
(dotfiles--key "u")
```

## Toggle Tangle on Save

Expose a setting for automatic tangling:

```elisp
(defcustom dotfiles-tangle-on-save t
  "Whether to tangle known Org dotfiles when saving them."
  :type 'boolean
  :group 'dotfiles)
```

This lets users keep `dotfiles-mode` keybindings enabled while disabling automatic tangling.

## Update Tangling to Link Files

From earlier episodes, `dotfiles-tangle-org-file` tangles one Org file.

After tangling, call the linker:

```elisp
(defun dotfiles-tangle-org-file (&optional org-file)
  "Tangle ORG-FILE.

When called interactively, prompt for ORG-FILE."
  (interactive "FOrg file: ")
  (let ((inhibit-message t)
        (message-log-max nil)
        (org-confirm-babel-evaluate nil))
    (org-babel-tangle-file
     (expand-file-name org-file dotfiles-folder)))
  (dotfiles-link-config-files))
```

This is simple, but not perfect.

It links all config files after every tangle. A more efficient implementation would link only the files generated by the current Org file. That requires more tracking, so the broad update is a good first version.

## Org Mode Hook

When an Org buffer opens, `dotfiles-mode` can add a buffer-local `after-save-hook`.

```elisp
(defun dotfiles--org-mode-hook ()
  "Set up Dotfiles behavior in Org buffers."
  (add-hook 'after-save-hook
            #'dotfiles--after-save-handler
            nil
            t))
```

The final `t` makes the hook buffer-local. That means the save handler is added only to the Org buffer that just opened.

## After-Save Handler

The save handler decides whether the current buffer should be tangled.

```elisp
(defun dotfiles--after-save-handler ()
  "Tangle the current Org file when appropriate."
  (when (and dotfiles-mode
             dotfiles-tangle-on-save
             buffer-file-name
             (member (file-name-nondirectory buffer-file-name)
                     dotfiles-org-files)
             (string=
              (file-truename
               (file-name-directory buffer-file-name))
              (file-truename
               (file-name-as-directory dotfiles-folder))))
    (message "Tangling %s"
             (file-name-nondirectory buffer-file-name))
    (dotfiles-tangle-org-file buffer-file-name)))
```

The checks matter.

`dotfiles-mode` must be enabled:

```elisp
dotfiles-mode
```

Auto-tangling must be enabled:

```elisp
dotfiles-tangle-on-save
```

The current buffer must be visiting a file:

```elisp
buffer-file-name
```

The file name must be one of the configured Org files:

```elisp
(member (file-name-nondirectory buffer-file-name)
        dotfiles-org-files)
```

And the file must live in the dotfiles folder:

```elisp
(string=
 (file-truename
  (file-name-directory buffer-file-name))
 (file-truename
  (file-name-as-directory dotfiles-folder)))
```

This prevents an unrelated file named `emacs.org` somewhere else from being tangled accidentally.

## The Real Minor Mode

Now define the full mode:

```elisp
(define-minor-mode dotfiles-mode
  "Toggle Dotfiles mode.

When enabled, known Org dotfiles can be tangled automatically
after saving, and Dotfiles commands are available under
`dotfiles-key-prefix'."
  :init-value nil
  :global t
  :group 'dotfiles
  :lighter " Dotfiles"
  :keymap `((,(dotfiles--key "t") . dotfiles-tangle-org-file)
            (,(dotfiles--key "u") . dotfiles-update))
  (if dotfiles-mode
      (add-hook 'org-mode-hook #'dotfiles--org-mode-hook)
    (remove-hook 'org-mode-hook #'dotfiles--org-mode-hook)))
```

When the mode turns on, it attaches `dotfiles--org-mode-hook` to `org-mode-hook`.

When the mode turns off, it removes that hook.

Any Org buffers that already received a buffer-local `after-save-hook` may still have that hook installed, so the save handler also checks `dotfiles-mode` before doing work. That extra check makes disabling the mode safe.

## Keybindings

With the default prefix:

```text
C-c C-.
```

the mode provides:

```text
C-c C-. t  dotfiles-tangle-org-file
C-c C-. u  dotfiles-update
```

Because these bindings live in the minor mode keymap, they are active only while `dotfiles-mode` is enabled.

Users can customize the prefix before loading the mode:

```elisp
(setq dotfiles-key-prefix (kbd "C-c d"))
```

or through the customization UI if the package exposes the customization group properly.

## Testing the Behavior

After evaluating the mode, enable it:

```elisp
(dotfiles-mode 1)
```

Open a configured Org file such as:

```text
~/.dotfiles/emacs.org
```

Make a change and save it.

If the file is listed in `dotfiles-org-files`, the save handler should tangle it and then update links.

Open another Org file that is not listed, such as:

```text
~/.dotfiles/README.org
```

Saving it should not tangle.

Disable automatic tangling:

```elisp
(setq dotfiles-tangle-on-save nil)
```

Saving a configured Org file should no longer tangle.

Re-enable it:

```elisp
(setq dotfiles-tangle-on-save t)
```

Disable the mode:

```elisp
(dotfiles-mode -1)
```

Saving should no longer tangle even if `dotfiles-tangle-on-save` is `t`.

## What to Remember

The key ideas:

- A major mode is the primary mode for a buffer.
- Minor modes add optional behavior.
- A minor mode needs state, a command, and optionally a keymap, lighter, and hooks.
- `minor-mode-alist` controls mode-line lighters.
- `minor-mode-map-alist` controls minor-mode keymaps.
- `run-hooks` runs ordinary hooks.
- Use `run-mode-hooks` for major modes.
- `define-minor-mode` handles the normal minor-mode conventions for you.
- Global minor modes can use `:global t`.
- Minor mode keymaps keep package bindings scoped.
- Buffer-local hooks are useful when global modes need per-buffer behavior.
- Save handlers should check enough context before doing work.

The dotfiles package now has a real user-facing mode. It can be enabled, disabled, customized, and used as the control point for automatic behavior. That is a big step from "a pile of helper functions" toward "an Emacs package."
