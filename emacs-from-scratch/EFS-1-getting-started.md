# Building an Emacs Configuration From Scratch

This tutorial walks through creating a small but useful Emacs configuration from an empty `init.el`. The goal is not to recreate a full framework like Doom Emacs or Spacemacs. Instead, you will build enough of the foundation yourself to understand how Emacs configuration works and why each piece matters.

By the end, you will have:

- a cleaner Emacs startup screen
- a readable font configuration
- a dark theme
- package installation through MELPA
- `use-package` for organizing package configuration
- `diminish` for hiding noisy minor modes
- Ivy and Counsel for better command, file, and buffer completion
- a nicer mode line with `doom-modeline`

## Why Configure Emacs Yourself?

Emacs can be used as a simple text editor, but it is more useful to think of it as a programmable productivity environment. You can use it for editing code, managing projects, reading documentation, working with Git, organizing notes, and even managing windows on Linux.

Prebuilt configurations like Doom Emacs and Spacemacs are valuable because they give you a polished setup quickly. The tradeoff is that they hide many of the decisions from you. Building a configuration from scratch teaches you how Emacs starts, how packages are loaded, how keybindings work, and how to change behavior when something does not fit your workflow.

This tutorial starts small on purpose. A good Emacs configuration grows over time.

## Prerequisites

You will need:

- Emacs installed
- basic comfort editing text files
- internet access for downloading packages

The original walkthrough used Emacs 27.1 on Linux, but the basic ideas apply on macOS and Windows too. Some details, such as fonts and window-manager integration, may differ by operating system.

## Start With a Clean Emacs Session

Before editing your normal Emacs configuration, it is useful to launch Emacs without loading your existing setup. This gives you a controlled environment where you can see exactly what each line of configuration changes.

Create an empty `init.el` file somewhere you can experiment:

```bash
touch init.el
```

Then start Emacs using that file:

```bash
emacs -q -l init.el
```

The `-q` flag tells Emacs not to load your usual user configuration. The `-l init.el` part tells Emacs to load the experimental file instead.

When Emacs opens, it will probably look plain: a white background, a toolbar, a menu bar, a startup screen, and maybe text that is too small. That is normal. The default interface is intentionally generic, and your configuration is where you turn it into your own environment.

## Simplify the Interface

The first thing to do is remove visual elements that are not necessary for a keyboard-driven editing workflow.

Add this to `init.el`:

```elisp
(setq inhibit-startup-message t)

(scroll-bar-mode -1)
(tool-bar-mode -1)
(tooltip-mode -1)
(menu-bar-mode -1)

(setq visible-bell t)
```

Here is what each line is doing:

- `inhibit-startup-message` skips the default welcome screen.
- `scroll-bar-mode -1` removes the graphical scroll bar.
- `tool-bar-mode -1` removes the icon toolbar.
- `tooltip-mode -1` disables graphical tooltips.
- `menu-bar-mode -1` removes the top menu bar.
- `visible-bell` replaces audible beeps with a visual flash.

This matters because Emacs becomes easier to understand when you remove visual noise. You are left with the actual editing surface, the mode line, and the minibuffer.

Reload the configuration by restarting Emacs with the same command:

```bash
emacs -q -l init.el
```

You should now land directly in the scratch buffer with a much simpler interface.

## Set a Readable Font

The default font may be too small, especially on a high-DPI display. Font configuration is personal, so treat this as a starting point rather than a rule.

Add this to `init.el`:

```elisp
(set-face-attribute 'default nil
                    :font "Fira Code Retina"
                    :height 280)
```

This changes the default face, which is the base font style Emacs uses for normal text.

The `:font` value should match a font installed on your system. If you do not have Fira Code installed, use another monospace font, such as:

```elisp
(set-face-attribute 'default nil
                    :font "DejaVu Sans Mono"
                    :height 140)
```

The `:height` value is in tenths of a point. A value of `140` means roughly 14pt. In the original video, the value was much larger because of the speaker's display scaling.

This step matters because readability is not cosmetic. If you live in an editor all day, font size and shape affect comfort, focus, and fatigue.

## Evaluate Changes Without Restarting

At first, restarting Emacs after every change is useful because it proves your config works from a clean start. Once you are editing the config from inside Emacs, you can evaluate it directly.

Open `init.el`, then run:

```text
M-x eval-buffer
```

In Emacs notation, `M-x` usually means `Alt-x`. It opens the command prompt where you can run interactive Emacs commands.

`eval-buffer` reads the current buffer as Emacs Lisp and applies it to the running Emacs session. This is useful while experimenting because you can make a change, evaluate it, and immediately see the result.

## Load a Dark Theme

The default white background can be harsh. Emacs includes several built-in themes, so you can load one without installing anything.

Add this to `init.el`:

```elisp
(load-theme 'wombat)
```

The quote before `wombat` matters. `load-theme` expects a symbol, not a string. In Emacs Lisp, `'wombat` means "use the symbol named wombat."

You may also see examples like this:

```elisp
(load-theme 'tango-dark)
```

Themes are partly aesthetic, but they also affect how easily you can scan code. Pick one that gives you enough contrast without becoming distracting.

## Learn the Help System Early

Emacs is self-documenting. That is one of its biggest strengths.

Two commands are especially useful:

```text
C-h f
C-h v
```

`C-h f` runs `describe-function`. Use it when you want to know what a function does, what arguments it takes, or where it is defined.

`C-h v` runs `describe-variable`. Use it when you want to inspect a variable, see its current value, or read its documentation.

For example, if you are unsure how `load-theme` works, run:

```text
C-h f load-theme
```

This gives you documentation inside Emacs itself. When configuring Emacs, this is often faster and more accurate than searching the web.

## Set Up Package Archives

Emacs has a built-in package manager. To use community packages, you need to tell Emacs where to find them.

Add this to `init.el`:

```elisp
(require 'package)

(setq package-archives
      '(("melpa" . "https://melpa.org/packages/")
        ("elpa" . "https://elpa.gnu.org/packages/")))

(package-initialize)

(unless package-archive-contents
  (package-refresh-contents))
```

Here is the purpose of each part:

- `(require 'package)` loads Emacs' package-management library.
- `package-archives` defines the package repositories Emacs can use.
- MELPA provides a large collection of community packages.
- ELPA is the official GNU Emacs package archive.
- `package-initialize` prepares installed packages for use.
- `package-refresh-contents` downloads the current package list if Emacs does not already have it.

This matters because a fresh Emacs install does not automatically know what is available in MELPA. Refreshing the archive contents gives Emacs a package index it can search and install from.

## Install and Load `use-package`

Emacs packages can be configured manually, but the configuration gets repetitive quickly. `use-package` gives you a clean, consistent way to install, load, bind, and configure packages.

Add this after the package setup:

```elisp
(unless (package-installed-p 'use-package)
  (package-install 'use-package))

(require 'use-package)

(setq use-package-always-ensure t)
```

The predicate function `package-installed-p` checks whether a package is already installed. In Emacs Lisp, functions ending in `-p` usually ask a yes-or-no question and return true or nil.

The `use-package-always-ensure` setting tells `use-package` to install packages automatically if they are missing. That is helpful when you move your config to a new machine because evaluating your config can pull down the packages it needs.

After adding this, run:

```text
M-x eval-buffer
```

On the first run, Emacs may contact MELPA and ELPA, download package metadata, and install `use-package`.

## Install `diminish` for Cleaner Mode Names

Minor modes often add their names to the mode line. That can be useful at first, but it gets noisy once your configuration grows.

Install `diminish` so `use-package` can hide minor modes you do not need to see constantly:

```elisp
(use-package diminish)
```

This does not disable the modes. It only removes their labels from the mode line when you choose to diminish them.

## Check Messages When Something Happens Too Fast

Sometimes Emacs flashes a message in the minibuffer and it disappears before you can read it. Those messages are saved in a special buffer.

Open it with:

```text
C-h e
```

Or switch to the buffer named:

```text
*Messages*
```

This buffer is useful when installing packages, debugging config problems, or checking whether a command did what you expected.

## Add Escape as a General Cancel Key

If you are used to Vim, you may instinctively press `Escape` to cancel prompts. Emacs often uses `C-g` for that behavior, but you can make `Escape` work too.

Add this:

```elisp
(global-set-key (kbd "<escape>") 'keyboard-escape-quit)
```

This binds the Escape key to `keyboard-escape-quit`, which cancels many active prompts or transient states.

This is a small change, but it makes Emacs feel less hostile if your muscle memory expects Escape to mean "get me out of this."

## Install Ivy and Counsel

Emacs has built-in completion, but many users prefer a richer completion interface. Ivy is a completion framework, and Counsel provides improved versions of common Emacs commands that use Ivy.

Add this to `init.el`:

```elisp
(use-package ivy
  :diminish
  :bind (("C-s" . swiper)
         :map ivy-minibuffer-map
         ("C-j" . ivy-next-line)
         ("C-k" . ivy-previous-line))
  :config
  (ivy-mode 1))

(use-package counsel
  :after ivy
  :config
  (counsel-mode 1))
```

This is the first larger `use-package` form, so break it down:

- `use-package ivy` declares configuration for the Ivy package.
- `:diminish` hides Ivy's mode name from the mode line.
- `:bind` defines keybindings.
- `C-s` is bound to `swiper`, an Ivy-powered search command.
- The `:map ivy-minibuffer-map` bindings only apply while Ivy's minibuffer is active.
- `C-j` and `C-k` move down and up in Ivy results.
- `:config` contains code that runs after the package is loaded.
- `(ivy-mode 1)` enables Ivy globally.

Counsel is loaded after Ivy and then enabled with:

```elisp
(counsel-mode 1)
```

This step matters because completion is one of the most common interactions in Emacs. You use it when opening files, switching buffers, running commands, searching, and selecting options. Better completion makes the whole editor feel faster.

Try these commands after evaluating the buffer:

```text
C-x C-f
C-x b
M-x
C-s
```

You should now see Ivy-style completion for common workflows.

## Understand `M-x`

`M-x` is one of the most important Emacs keybindings. It lets you run interactive commands by name.

With Ivy and Counsel enabled, `M-x` becomes much easier to use because you can type fragments of a command and fuzzy-match the result. For example, typing something like:

```text
switch buff
```

can help you find a buffer-switching command even if you do not remember the exact name.

This is important because Emacs has thousands of commands. You do not need to memorize all of them. You need a good way to discover and run them.

## Use Swiper for Searching

The Ivy configuration above binds `C-s` to `swiper`.

Swiper searches the current buffer and shows matching lines in an interactive completion list. This is more informative than the default incremental search because you can see multiple matches at once and jump directly to the one you want.

Try:

```text
C-s
```

Then type a word that appears in the current file. Use `C-j` and `C-k` to move through the results if you used the bindings from the previous step.

This is a good example of why completion frameworks matter: they improve many small daily interactions, not just one command.

## Clean Up the Mode Line

The mode line is the status bar at the bottom of each Emacs window. It shows information such as the current buffer, major mode, minor modes, line number, and version-control state.

The default mode line is functional but visually busy. `doom-modeline` gives you a cleaner and more modern version.

Add this:

```elisp
(use-package doom-modeline
  :init
  (doom-modeline-mode 1)
  :custom
  (doom-modeline-height 15))
```

The `:init` section runs before the package is fully loaded. Here, it enables `doom-modeline-mode`.

The `:custom` section sets package variables using Emacs' customization system. In this case, it sets the modeline height.

This step matters because the mode line is always visible. If it is cluttered, it becomes constant visual noise. A cleaner mode line makes Emacs feel calmer and easier to scan.

## Optional: Show Commands While Teaching

If you are recording tutorials, pairing, or showing someone your Emacs setup, it can be useful to display the keys you press and the commands they run.

Install `command-log-mode`:

```elisp
(use-package command-log-mode)
```

Then run:

```text
M-x global-command-log-mode
M-x clm/toggle-command-log-buffer
```

This opens a buffer that displays recent keypresses and commands.

This is not necessary for everyday editing, but it is useful when teaching because viewers can see what you are doing without needing every keybinding narrated out loud.

## Full Example `init.el`

Here is the complete configuration from this tutorial:

```elisp
(setq inhibit-startup-message t)

(scroll-bar-mode -1)
(tool-bar-mode -1)
(tooltip-mode -1)
(menu-bar-mode -1)

(setq visible-bell t)

(set-face-attribute 'default nil
                    :font "Fira Code Retina"
                    :height 280)

(load-theme 'wombat)

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

(use-package diminish)

(global-set-key (kbd "<escape>") 'keyboard-escape-quit)

(use-package ivy
  :diminish
  :bind (("C-s" . swiper)
         :map ivy-minibuffer-map
         ("C-j" . ivy-next-line)
         ("C-k" . ivy-previous-line))
  :config
  (ivy-mode 1))

(use-package counsel
  :after ivy
  :config
  (counsel-mode 1))

(use-package doom-modeline
  :init
  (doom-modeline-mode 1)
  :custom
  (doom-modeline-height 15))

(use-package command-log-mode)
```

If the font line fails, replace `"Fira Code Retina"` with a font installed on your system and lower the height value if the text is too large.

## What to Learn Next

This configuration is intentionally small. It gives you the foundation for a more serious setup without hiding the mechanics.

Good next packages to explore are:

- `magit` for Git
- `projectile` for project navigation
- `which-key` for keybinding discovery
- `evil` for Vim-style editing
- `company` or `corfu` for completion at point
- language-specific modes for the languages you use

The important habit is to add one package at a time and understand why it is there. Emacs can become a complete computing environment, but it gets there through small, understandable pieces.
