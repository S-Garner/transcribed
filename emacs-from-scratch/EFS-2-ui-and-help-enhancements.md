# Emacs From Scratch 2: UI and Help Enhancements

In the first part, you built a small Emacs configuration that made the default interface cleaner and added a few core packages: `use-package`, Ivy, Counsel, Doom Modeline, and a theme.

This part builds on that foundation. The goal is to make Emacs easier to understand while you are using it. That means adding better visual context, better keybinding discovery, better command descriptions, and better help buffers.

By the end, you will have:

- line numbers in normal editing buffers
- column numbers in the mode line
- line numbers disabled in terminal-style buffers
- a basic understanding of modes and hooks
- rainbow delimiters for programming buffers
- Which Key for keybinding discovery
- Ivy Rich for richer completion metadata
- Helpful for improved help buffers
- Doom Themes for better-looking and better-integrated themes

## Starting Point

This tutorial assumes you already have the basic configuration from part 1, including package setup and `use-package`.

At minimum, your config should already include something like this:

```elisp
(require 'package)

(setq package-archives
      '(("melpa" . "https://melpa.org/packages/")
        ("elpa" . "https://elpa.gnu.org/packages/")))

(package-initialize)

(unless package-archive-contents
  (package-refresh-contents))

(unless (package-installed-p 'use-package)
  (package-install 'use-package))

(require 'use-package)

(setq use-package-always-ensure t)
```

That setup matters because most of this tutorial uses community packages from MELPA. `use-package-always-ensure` tells Emacs to install missing packages automatically when it evaluates your config.

## Show Line Numbers

Line numbers are useful when editing source files because they give you a stable reference point. They help when reading compiler errors, discussing code with another person, or jumping to a specific location.

Modern Emacs has built-in line number support, so you do not need an external package.

Add this to your `init.el`:

```elisp
(global-display-line-numbers-mode 1)
```

The `global-` prefix means this applies broadly across buffers. After evaluating that expression, most buffers will show line numbers in the left margin.

To evaluate only this expression, place your cursor after the closing parenthesis and run:

```text
C-x C-e
```

That command evaluates the Emacs Lisp expression before point. It is more targeted than evaluating the entire buffer.

## Show Column Numbers

The mode line can show your current line number, but it does not always show your column number by default. Column numbers are helpful when working with languages or tools that report exact line and column positions.

Add this:

```elisp
(column-number-mode 1)
```

Now the mode line shows both line and column position.

This is a small UI improvement, but it pays off whenever you are matching an error message to a location in a file.

## Disable Line Numbers in Terminal Buffers

Global line numbers are useful in source files, but they are noisy in terminal-like buffers. If you run a shell inside Emacs, line numbers usually do not add meaningful information.

To handle that, disable line numbers in specific modes:

```elisp
(dolist (mode '(term-mode-hook
                shell-mode-hook
                eshell-mode-hook))
  (add-hook mode (lambda () (display-line-numbers-mode 0))))
```

This snippet introduces two important Emacs ideas: modes and hooks.

## What Modes Are

A mode is a bundle of behavior for a buffer.

For example:

- `emacs-lisp-mode` gives Emacs Lisp files syntax highlighting, indentation, and editing behavior.
- `shell-mode` gives shell buffers shell-specific behavior.
- `term-mode` gives terminal buffers terminal-specific behavior.
- `prog-mode` is a general parent mode for programming-language modes.

Most buffers have one major mode. A major mode defines the main behavior of that buffer. A buffer can also have multiple minor modes enabled on top of that.

This matters because most Emacs customization is attached to modes. Instead of saying "always do this everywhere," you often say "do this when this kind of buffer opens."

## What Hooks Are

A hook is a variable that stores functions to run at a specific time.

Mode hooks run when a mode starts. For example, `shell-mode-hook` runs when a buffer enters `shell-mode`.

This line:

```elisp
(add-hook 'shell-mode-hook (lambda () (display-line-numbers-mode 0)))
```

means:

"When `shell-mode` starts, run this anonymous function, and that function turns off line numbers in this buffer."

The tutorial version uses `dolist` so the same behavior can be attached to several hooks without repeating the same code.

One thing to know while experimenting: if you evaluate an `add-hook` expression repeatedly, you can accidentally add the same hook function more than once. That is usually not a big problem while learning, but it explains why targeted evaluation is often better than repeatedly running the whole config.

## Inspect Hooks With Help

Emacs can show you what is attached to a hook.

Run:

```text
C-h v shell-mode-hook
```

`C-h v` runs `describe-variable`. Since hooks are variables, this shows the hook's current value and documentation.

You can also evaluate a variable directly with:

```text
M-:
```

Then type:

```elisp
shell-mode-hook
```

`M-:` opens an eval prompt. This is useful when you want to inspect a value quickly without opening a full help buffer.

## Add Rainbow Delimiters

Emacs is configured with Emacs Lisp, and Lisp code contains a lot of parentheses. That can look intimidating at first, but good editor support makes it much easier to read.

`rainbow-delimiters` colors nested parentheses differently, so you can visually match opening and closing delimiters.

Add this:

```elisp
(use-package rainbow-delimiters
  :hook (prog-mode . rainbow-delimiters-mode))
```

The `:hook` line says:

"Whenever `prog-mode` is active, enable `rainbow-delimiters-mode`."

Since many programming modes derive from `prog-mode`, this turns on rainbow delimiters in Emacs Lisp, JavaScript, Python, and many other programming buffers.

You could make it narrower:

```elisp
(use-package rainbow-delimiters
  :hook (emacs-lisp-mode . rainbow-delimiters-mode))
```

That would enable it only for Emacs Lisp. The broader `prog-mode` version is a reasonable default because delimiters matter in many programming languages.

## Add Which Key

Emacs has many keybindings, and some of the most useful ones are hidden behind prefixes such as `C-h` and `C-x`.

Which Key helps by showing a popup when you start a key sequence. It tells you which keys are available next and what commands they run.

Add this:

```elisp
(use-package which-key
  :init
  (which-key-mode)
  :custom
  (which-key-idle-delay 0.3))
```

Now press:

```text
C-h
```

After a short delay, Which Key displays the available bindings under the `C-h` prefix. You should see entries such as:

- `f` for `describe-function`
- `v` for `describe-variable`
- `k` for `describe-key`
- `m` for `describe-mode`

This matters because keybinding discovery becomes interactive. You do not need to memorize every binding before you can use Emacs effectively.

## Understand Common Prefixes

Two prefixes are especially important:

```text
C-h
C-x
```

`C-h` is the help prefix. Use it when you want Emacs to explain something.

`C-x` is a general command prefix. Many common editing, file, buffer, and window commands live there.

For example:

```text
C-x C-f
```

opens a file.

```text
C-x C-s
```

saves the current buffer.

```text
C-x 0
```

closes the current Emacs window.

```text
C-x 1
```

closes other Emacs windows and keeps the current one.

Which Key makes these prefixes much less mysterious because it shows the available choices as you type.

## Tune Which Key's Delay

The `which-key-idle-delay` value controls how quickly the popup appears.

This shows it almost immediately:

```elisp
(setq which-key-idle-delay 0.0)
```

This waits one second:

```elisp
(setq which-key-idle-delay 1.0)
```

A short delay is helpful while learning. A longer delay may feel better once you already know your bindings and only want help when you pause.

If changing the delay does not appear to take effect immediately, toggle Which Key off and back on:

```text
M-x which-key-mode
M-x which-key-mode
```

## Add Ivy Rich

Ivy and Counsel improve completion, but by default many completion lists only show names. Ivy Rich adds useful metadata to Ivy completion results.

For example, when running `M-x`, Ivy Rich can show:

- the command name
- the keybinding for that command, if one exists
- the command's documentation summary

Add this:

```elisp
(use-package ivy-rich
  :init
  (ivy-rich-mode 1))
```

Now run:

```text
M-x
```

If `M-x` is bound to `counsel-M-x`, you should see richer command information.

This matters because command discovery gets much better. You can search for commands by name and see what they do before running them.

## Bind Counsel Commands Explicitly

To make sure the richer Counsel versions are used for common commands, configure Counsel bindings directly.

Add or update your Counsel configuration:

```elisp
(use-package counsel
  :after ivy
  :bind (("M-x" . counsel-M-x)
         ("C-x b" . counsel-ibuffer)
         ("C-x C-f" . counsel-find-file)
         :map minibuffer-local-map
         ("C-r" . counsel-minibuffer-history))
  :config
  (counsel-mode 1))
```

Here is what these bindings do:

- `M-x` uses `counsel-M-x`, giving you an Ivy-powered command list.
- `C-x b` uses `counsel-ibuffer`, giving you a better buffer switcher.
- `C-x C-f` uses `counsel-find-file`, giving you better file navigation.
- `C-r` in the minibuffer opens minibuffer history.

The minibuffer history binding is useful when you are repeating or editing previous minibuffer inputs, such as command names, file paths, or search terms.

## Use Ivy Actions With `M-o`

Ivy completion lists often provide extra actions for the selected item.

While an Ivy list is open, press:

```text
M-o
```

This opens an action menu.

For example, in `counsel-M-x`, selecting a command and pressing `M-o` may let you:

- jump to the command definition
- open help for the command
- run another context-specific action

This matters because Ivy is not just a picker. It can also be a contextual action menu. When you find the thing you want, `M-o` often lets you decide what to do with it.

## Add Helpful

Emacs' built-in help is already good, but the `helpful` package makes it easier to read and more informative.

Helpful can show:

- better-formatted documentation
- keybindings for a function
- the source code of a function
- where a variable or function is defined
- where a variable is referenced
- links to related documentation

Add this:

```elisp
(use-package helpful
  :custom
  (counsel-describe-function-function #'helpful-callable)
  (counsel-describe-variable-function #'helpful-variable)
  :bind
  ([remap describe-function] . counsel-describe-function)
  ([remap describe-command] . helpful-command)
  ([remap describe-variable] . counsel-describe-variable)
  ([remap describe-key] . helpful-key))
```

This block does two things.

First, it tells Counsel to use Helpful when describing functions and variables:

```elisp
(counsel-describe-function-function #'helpful-callable)
(counsel-describe-variable-function #'helpful-variable)
```

Second, it remaps existing help commands.

The `remap` form is useful because it changes what an existing command binding does without requiring you to know or duplicate the exact keybinding. For example, `C-h f` normally runs `describe-function`. This remap makes that same keybinding use the Counsel and Helpful version instead.

Try:

```text
C-h f describe-function
```

You should get a richer Helpful buffer instead of the default help buffer.

Try a variable too:

```text
C-h v package-user-dir
```

Helpful shows the current value, original value, documentation, source definition, and references where available.

This is one of the most useful packages for learning Emacs because it turns Emacs itself into a better reference manual.

## Add Doom Themes

Built-in Emacs themes are fine, but they are usually designed around base Emacs. Larger packages such as Magit, Org Mode, Ivy, and Doom Modeline often look better with themes that explicitly support them.

`doom-themes` provides a large set of themes originally associated with Doom Emacs, and they generally integrate well with common Emacs packages.

Add this:

```elisp
(use-package doom-themes
  :init
  (load-theme 'doom-gruvbox t))
```

The `t` tells `load-theme` not to ask for confirmation.

You can replace `doom-gruvbox` with another Doom theme, such as:

```elisp
(load-theme 'doom-nord t)
(load-theme 'doom-palenight t)
(load-theme 'doom-dracula t)
(load-theme 'doom-tomorrow-night t)
```

Only load one theme by default in your config. The list above shows alternatives, not code you should paste all at once.

## Preview Themes With Counsel

Instead of guessing theme names, use Counsel to browse available themes:

```text
M-x counsel-load-theme
```

This opens a completion list of installed themes. Move through the list to preview them.

When choosing a theme, do not judge only by whether it looks stylish in one buffer. Check a few different file types:

- Emacs Lisp
- a language you use for work
- Org Mode, if you use it
- Magit, once you install it
- shell or terminal buffers

A theme can look great in one mode and be hard to read in another. Contrast matters more than novelty when you spend hours in the editor.

## Full Example Configuration Additions

Here are the additions from this part in one block:

```elisp
(global-display-line-numbers-mode 1)
(column-number-mode 1)

(dolist (mode '(term-mode-hook
                shell-mode-hook
                eshell-mode-hook))
  (add-hook mode (lambda () (display-line-numbers-mode 0))))

(use-package rainbow-delimiters
  :hook (prog-mode . rainbow-delimiters-mode))

(use-package which-key
  :init
  (which-key-mode)
  :custom
  (which-key-idle-delay 0.3))

(use-package ivy-rich
  :init
  (ivy-rich-mode 1))

(use-package counsel
  :after ivy
  :bind (("M-x" . counsel-M-x)
         ("C-x b" . counsel-ibuffer)
         ("C-x C-f" . counsel-find-file)
         :map minibuffer-local-map
         ("C-r" . counsel-minibuffer-history))
  :config
  (counsel-mode 1))

(use-package helpful
  :custom
  (counsel-describe-function-function #'helpful-callable)
  (counsel-describe-variable-function #'helpful-variable)
  :bind
  ([remap describe-function] . counsel-describe-function)
  ([remap describe-command] . helpful-command)
  ([remap describe-variable] . counsel-describe-variable)
  ([remap describe-key] . helpful-key))

(use-package doom-themes
  :init
  (load-theme 'doom-gruvbox t))
```

If you already have a `use-package counsel` block from part 1, merge the new bindings into that existing block instead of keeping two separate Counsel configurations.

## What This Gives You

This part is mostly about visibility.

Line numbers and column numbers make buffers easier to navigate. Hooks teach you how to customize behavior for specific kinds of buffers. Rainbow delimiters make Lisp and other delimiter-heavy languages easier to read. Which Key helps you discover keybindings as you type. Ivy Rich gives more context in completion lists. Helpful makes Emacs' documentation more useful. Doom Themes improves the visual polish across more packages.

None of these packages radically changes how you edit yet. Instead, they make Emacs easier to inspect, learn, and extend. That is the right next step before adding larger workflow packages such as Magit, Projectile, Evil, or language-server support.
