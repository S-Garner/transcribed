# Emacs From Scratch 3: Keybindings and Evil Mode

In the first two parts, you built a minimal Emacs configuration and added UI helpers that make Emacs easier to inspect. This part focuses on keybindings.

Keybindings are central to why many people use Emacs. Emacs lets you control nearly everything from the keyboard, and its configuration system lets you decide what those keys should do. In this tutorial, you will start with basic Emacs keybinding functions, then move into more structured keybinding tools: `general.el`, Evil mode, Evil Collection, and Hydra.

By the end, you will have:

- fixed two small issues from the earlier setup
- learned how `global-set-key` works
- learned how mode-specific keymaps work
- used `general.el` to define cleaner keybindings
- added a leader-key style prefix
- enabled Vim-style editing with Evil mode
- added Evil bindings for more Emacs modes with Evil Collection
- created a transient key menu with Hydra

## Fix Icon Fonts for Doom Modeline

If you added `doom-modeline` earlier, you may have noticed that some icons did not render correctly. That happens because Doom Modeline depends on icon fonts, and those fonts need to be installed on your system.

Add this near your Doom Modeline setup:

```elisp
(use-package all-the-icons)
```

Then run this command once:

```text
M-x all-the-icons-install-fonts
```

This installs the icon fonts into your operating system's font location so Emacs can use them.

This matters because the package installation alone is not enough. `all-the-icons` provides the Emacs package, but the actual font files also need to exist where your system can find them.

## Fix Theme Loading Prompts

If your config loads a theme like this:

```elisp
(load-theme 'doom-gruvbox)
```

Emacs may ask for confirmation every time it starts:

```text
Loading a theme can run Lisp code. Really load?
```

That prompt is useful when you are manually loading unknown themes, but it is annoying for a theme you intentionally load from your own config.

Change the call to:

```elisp
(load-theme 'doom-gruvbox t)
```

The second argument, `t`, tells Emacs not to ask for confirmation.

## Why Keybindings Matter

Emacs is designed to be driven from the keyboard. That is useful because keyboard commands are fast, repeatable, and easy to compose into a workflow.

This is especially valuable if you spend all day at a computer. Reaching for the mouse constantly can slow you down and can also be uncomfortable over long sessions. Emacs gives you the tools to keep more of your work on the keyboard.

If you have not already remapped Caps Lock to Control, it is worth considering. Many Emacs bindings use Control, and putting Control on the home row makes those bindings much easier on your hands.

## Set a Global Keybinding

The most direct way to bind a key globally is `global-set-key`.

For example, this binds `C-M-j` to `counsel-switch-buffer`:

```elisp
(global-set-key (kbd "C-M-j") 'counsel-switch-buffer)
```

Here is what is happening:

- `C` means Control.
- `M` means Meta, which is usually Alt on modern keyboards.
- `j` is the letter key.
- `(kbd "C-M-j")` converts the human-readable key string into the internal key representation Emacs expects.
- `counsel-switch-buffer` is the command that runs when the key is pressed.

After evaluating that expression, pressing `C-M-j` opens an Ivy/Counsel buffer switcher.

This matters because global bindings are available almost everywhere. Use them for commands you want to reach from any buffer.

## Unset a Global Keybinding

When experimenting, you may want to remove a binding you just added.

Run this from your config or from the `M-:` eval prompt:

```elisp
(global-unset-key (kbd "C-M-j"))
```

The `M-:` command opens a minibuffer prompt where you can evaluate Emacs Lisp directly. That is handy for quick experiments because you do not need to edit and re-evaluate your config file.

After unsetting the key, pressing `C-M-j` should report that the key is undefined.

## Set a Mode-Specific Keybinding

Global bindings are useful, but sometimes you only want a keybinding in one kind of buffer. For that, use a mode keymap.

Every major mode usually has a keymap variable. The naming convention is commonly:

```text
MODE-NAME-map
```

For example, Emacs Lisp mode uses:

```elisp
emacs-lisp-mode-map
```

You can inspect it with:

```text
C-h v emacs-lisp-mode-map
```

To bind a key only in Emacs Lisp buffers, use `define-key`:

```elisp
(define-key emacs-lisp-mode-map
  (kbd "C-x M-t")
  'counsel-load-theme)
```

This means `C-x M-t` will run `counsel-load-theme`, but only while `emacs-lisp-mode` is active.

This matters because local keybindings let the same key mean mode-specific things. A keybinding in an Emacs Lisp buffer can evaluate Lisp, while the same binding in a Python buffer could send code to a Python REPL.

## Use `general.el` for Cleaner Bindings

Emacs' built-in keybinding functions work, but larger configurations can become repetitive. `general.el` gives you a cleaner syntax for defining keys.

Install it:

```elisp
(use-package general)
```

Then define a binding:

```elisp
(general-define-key
  "C-M-j" 'counsel-switch-buffer)
```

The simple version is not dramatically different from `global-set-key`, but `general.el` becomes more useful when you define many keys, mode-specific keys, or a leader-key prefix.

## Create a Leader-Key Prefix

A leader key is a personal prefix where you put your own command map. Users of Spacemacs or Doom Emacs may recognize this idea from the Space key.

The benefit is organization. Instead of scattering custom bindings everywhere, you create one predictable place for your commands.

Add this:

```elisp
(use-package general
  :after evil
  :config
  (general-evil-setup)

  (general-create-definer rune/leader-keys
    :keymaps '(normal insert visual emacs)
    :prefix "SPC"
    :global-prefix "C-SPC"))
```

This creates a new function called `rune/leader-keys`.

The name `rune/leader-keys` is just a personal namespace. Emacs Lisp does not have namespaces in the way some languages do, so users often prefix their own functions with a name, initials, or project label.

The important parts are:

- `:prefix "SPC"` means Space acts as the leader in Evil states where that makes sense.
- `:global-prefix "C-SPC"` means `C-SPC` works globally.
- `:keymaps '(normal insert visual emacs)` says which Evil states should use the binding.

If you are not using Evil mode yet, this exact version may not work as expected. It is included here because the rest of this tutorial enables Evil.

The `(general-evil-setup)` call teaches General about Evil states such as `normal`, `insert`, `visual`, and `emacs`. Without that, General may not interpret the Evil-specific keymap shorthand correctly.

## Add Leader Bindings

Once the definer exists, you can call it like a function:

```elisp
(rune/leader-keys
  "t"  '(:ignore t :which-key "toggles")
  "tt" '(counsel-load-theme :which-key "choose theme"))
```

This creates a `t` prefix under your leader key and labels it as `toggles` for Which Key.

Then `tt` runs `counsel-load-theme`.

With the global prefix, the full key sequence is:

```text
C-SPC t t
```

If Evil is active and Space is available as a leader key, this also works as:

```text
SPC t t
```

This matters because the leader-key structure scales well. You can put Git commands under `g`, file commands under `f`, project commands under `p`, toggles under `t`, and so on.

## Why Use Evil Mode?

Evil mode is a Vim emulation layer for Emacs. It gives Emacs modal editing.

In normal Emacs editing, most editing commands use modifier keys such as Control and Meta. In Vim-style editing, you spend much of your time in normal mode, where regular letter keys run editing commands.

For example:

- `j` moves down
- `k` moves up
- `dd` deletes the current line
- `u` undoes
- `i` enters insert mode
- `Escape` returns to normal mode

This can reduce how often you need to hold modifier keys while editing text. You still get Emacs' ecosystem, but with Vim-style buffer editing.

## Install Evil Mode

Add this:

```elisp
(use-package evil
  :init
  (setq evil-want-integration t)
  (setq evil-want-keybinding nil)
  (setq evil-want-C-u-scroll t)
  (setq evil-want-C-i-jump nil)
  :config
  (evil-mode 1))
```

The `:init` block matters here. These variables need to be set before Evil loads because Evil reads them while setting itself up.

Here is what the variables mean:

- `evil-want-integration` enables Evil integration support.
- `evil-want-keybinding` is set to `nil` because Evil Collection will provide better mode-specific bindings later.
- `evil-want-C-u-scroll` makes `C-u` scroll up like Vim instead of running Emacs' `universal-argument`.
- `evil-want-C-i-jump` disables Evil's `C-i` jump behavior if you want to use that key for something else.

If you rely heavily on Emacs' `universal-argument`, think carefully before enabling `evil-want-C-u-scroll`. The original walkthrough prefers Vim's `C-u` scrolling behavior.

## Refresh Package Archives if Install Fails

Sometimes installing a package fails with a message that looks like the package could not be found. Often that means your local package archive list is out of date.

Run:

```text
M-x list-packages
```

This refreshes the package list from your configured archives.

Then evaluate the `use-package` form again.

This is useful to know because package errors are not always configuration errors. Sometimes Emacs simply needs a fresh list of available packages.

## Basic Evil Editing

Once Evil mode is enabled, the mode line shows which Evil state you are in.

Common states include:

- normal state: command-oriented editing
- insert state: typing text into the buffer
- visual state: selecting text
- emacs state: temporarily using normal Emacs behavior

Try these keys in a normal text buffer:

```text
j
k
i
Escape
dd
u
V
```

The sequence demonstrates moving, entering insert mode, returning to normal mode, deleting a line, undoing, and selecting a line.

This is the point where Emacs starts feeling much more comfortable if you already know Vim.

## Make `C-g` Return to Normal State

In Emacs, `C-g` is the general "quit this" command. When something strange is happening, pressing `C-g` often gets you back to safety.

With Evil, it is useful to make `C-g` return to normal state from insert state:

```elisp
(define-key evil-insert-state-map (kbd "C-g") 'evil-normal-state)
```

Now, while inserting text, `C-g` puts you back in normal state.

This reinforces a useful habit: `C-g` means "get me out of the current interaction."

## Optional: Make `C-h` Delete Backward in Insert State

In traditional Emacs keybindings, `C-h` often behaves like Backspace in text-entry contexts. You can make that work in Evil insert state:

```elisp
(define-key evil-insert-state-map (kbd "C-h") 'evil-delete-backward-char-and-join)
```

The advantage is that Backspace becomes available from the home row.

The tradeoff is that `C-h` is also Emacs' help prefix. If you press `C-h` while in insert state expecting help, it will delete backward instead. You will need to return to normal state first, then press `C-h`.

This is a personal preference. Use it only if home-row Backspace matters more to you than having help available directly from insert state.

## Make `j` and `k` Respect Visual Lines

When long lines wrap visually, Vim-style `j` and `k` can feel surprising. By default, they may move by real lines rather than the wrapped visual lines you see on screen.

To make movement follow visual lines:

```elisp
(define-key evil-normal-state-map (kbd "j") 'evil-next-visual-line)
(define-key evil-normal-state-map (kbd "k") 'evil-previous-visual-line)
```

This is especially useful when writing prose, Markdown, or Org Mode documents where long lines wrap.

## Use Evil Window Bindings

Evil gives you Vim-style window management with the `C-w` prefix.

Useful examples:

```text
C-w v
C-w s
C-w c
```

These commands split windows vertically, split windows horizontally, and close the current window.

This matters because window management is something you do constantly in Emacs. The `C-w` bindings are compact and become muscle memory quickly if you have Vim experience.

## Add Evil Collection

Evil mode gives you Vim-style editing in normal buffers, but many special Emacs modes need their own bindings. Help buffers, package menus, Magit, Dired, and other interfaces are not plain text editing buffers.

Evil Collection provides Evil-friendly keybindings for many common Emacs modes.

Add this after Evil:

```elisp
(use-package evil-collection
  :after evil
  :config
  (evil-collection-init))
```

The `:after evil` part makes sure Evil loads first.

The `(evil-collection-init)` call enables the collection's mode-specific bindings.

One simple test is a help buffer. Open help with:

```text
C-h f evil-collection-init
```

Then press:

```text
q
```

With Evil Collection active, `q` should close the help buffer in the familiar Vim-like way.

## Adjust Evil Collection if a Mode Misbehaves

Evil Collection supports many modes. That is useful, but you may eventually find one mode where you do not like the provided bindings.

The package controls which modes it configures through a mode list variable. You can inspect it with:

```text
C-h v evil-collection-mode-list
```

If a specific mode causes trouble, you can customize that list before calling `evil-collection-init`.

You do not need to do that now. The important idea is that Evil Collection is configurable, so you are not stuck with every default forever.

## Add Hydra

Hydra lets you create temporary keymaps for repeated actions.

This is useful when you want to press one full keybinding once, then use short single-key commands repeatedly.

Install it:

```elisp
(use-package hydra)
```

Then define a Hydra for text scaling:

```elisp
(defhydra hydra-text-scale (:timeout 4)
  "scale text"
  ("j" text-scale-increase "in")
  ("k" text-scale-decrease "out")
  ("f" nil "finished" :exit t))
```

This creates a command named:

```elisp
hydra-text-scale/body
```

When you run that command, a small prompt appears. You can press:

- `j` to increase text scale
- `k` to decrease text scale
- `f` to finish

The `:timeout 4` option closes the Hydra if no input happens for four seconds.

This matters because scaling text is often repetitive. Without Hydra, you would need to run a full command or keybinding again and again. With Hydra, you enter a temporary mode and use single keys until you are done.

## Bind the Hydra Under Your Leader Key

If you created the leader-key definer earlier, bind the text-scale Hydra under `t`:

```elisp
(rune/leader-keys
  "t"  '(:ignore t :which-key "toggles")
  "tt" '(counsel-load-theme :which-key "choose theme")
  "ts" '(hydra-text-scale/body :which-key "scale text"))
```

Now you can run:

```text
C-SPC t s
```

Then press `j` and `k` repeatedly to change text size.

This is where General, Which Key, and Hydra work nicely together:

- General defines the leader-key structure.
- Which Key shows the available commands.
- Hydra gives you a temporary repeated-action menu.

## Full Example Configuration Additions

Here is the full set of additions from this part:

```elisp
(use-package all-the-icons)

(use-package evil
  :init
  (setq evil-want-integration t)
  (setq evil-want-keybinding nil)
  (setq evil-want-C-u-scroll t)
  (setq evil-want-C-i-jump nil)
  :config
  (evil-mode 1)
  (define-key evil-insert-state-map (kbd "C-g") 'evil-normal-state)
  (define-key evil-insert-state-map (kbd "C-h") 'evil-delete-backward-char-and-join)
  (define-key evil-normal-state-map (kbd "j") 'evil-next-visual-line)
  (define-key evil-normal-state-map (kbd "k") 'evil-previous-visual-line))

(use-package evil-collection
  :after evil
  :config
  (evil-collection-init))

(use-package general
  :after evil
  :config
  (general-evil-setup)

  (general-create-definer rune/leader-keys
    :keymaps '(normal insert visual emacs)
    :prefix "SPC"
    :global-prefix "C-SPC")

  (rune/leader-keys
    "t"  '(:ignore t :which-key "toggles")
    "tt" '(counsel-load-theme :which-key "choose theme")))

(use-package hydra)

(defhydra hydra-text-scale (:timeout 4)
  "scale text"
  ("j" text-scale-increase "in")
  ("k" text-scale-decrease "out")
  ("f" nil "finished" :exit t))

(rune/leader-keys
  "ts" '(hydra-text-scale/body :which-key "scale text"))
```

If your Doom theme still looks like this:

```elisp
(load-theme 'doom-gruvbox)
```

change it to:

```elisp
(load-theme 'doom-gruvbox t)
```

## What This Gives You

This part changes the feel of Emacs more than the earlier UI improvements did.

`global-set-key` and `define-key` teach you the built-in model: global bindings for everywhere, mode maps for specific contexts. `general.el` gives you a cleaner way to organize larger sets of bindings. Evil mode gives you Vim-style editing inside Emacs. Evil Collection extends that editing model into more special-purpose buffers. Hydra gives you temporary keymaps for repetitive tasks.

The main idea is not just "add Vim bindings." The bigger idea is that Emacs lets you design your own keyboard interface. Once you have a leader key, Which Key, Evil, and Hydra working together, you have a strong foundation for adding project commands, Git commands, navigation commands, and language-specific workflows later.
