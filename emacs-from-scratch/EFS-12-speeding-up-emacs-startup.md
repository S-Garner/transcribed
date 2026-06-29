# Emacs From Scratch 12: Speeding Up Emacs Startup

As your Emacs configuration grows, startup usually gets slower. Every package, hook, theme, helper mode, completion system, and language tool adds a little bit of work.

Emacs itself is not usually the slow part. A plain session starts quickly. The slowdown normally comes from your configuration loading more code than it needs during startup.

This tutorial walks through a practical process:

- measure startup time
- identify packages that load too early
- defer package loading with `use-package`
- delay loose configuration with `with-eval-after-load`
- tune garbage collection during startup

The goal is not to make every package lazy-loaded at all costs. The goal is to load only what you need immediately, then defer everything else until it is actually useful.

## Compare Plain Emacs Startup

Start Emacs without your configuration:

```bash
emacs -q
```

This launches plain Emacs without loading your init file.

If `emacs -q` starts quickly but your normal Emacs takes several seconds, the problem is your configuration, not Emacs itself.

That is good news. Configuration can be improved.

## Measure Startup Time

Emacs records startup timing in two variables:

```elisp
before-init-time
after-init-time
```

You can inspect the startup time for the current session with:

```elisp
(emacs-init-time)
```

For ongoing tuning, it is useful to print startup time automatically.

Add this near the top of your configuration:

```elisp
(defun efs/display-startup-time ()
  (message "Emacs loaded in %s with %d garbage collections."
           (format "%.2f seconds"
                   (float-time
                    (time-subtract after-init-time before-init-time)))
           gcs-done))

(add-hook 'emacs-startup-hook #'efs/display-startup-time)
```

On startup, Emacs will now print something like:

```text
Emacs loaded in 4.88 seconds with 23 garbage collections.
```

That gives you a baseline. Without a baseline, performance tuning turns into vibes and suspicious staring.

## Understand What Needs to Load Early

The best startup optimization is simple:

Load less during startup.

Some things really do need to happen early:

- package manager setup
- `use-package` setup
- basic UI settings
- font configuration
- themes and mode line setup
- core keybinding packages, if the rest of your config depends on them

Other things usually do not need to load immediately:

- help packages
- Git interfaces
- LSP and debugging tools
- language modes
- terminal modes
- file-manager extensions
- Org extras, unless you need them at startup

The work is to look at every `use-package` block and ask: does this package need to be loaded before I can use Emacs, or can it wake up later?

## How `use-package` Defers Loading

`use-package` has several keywords that can delay loading automatically.

### `:hook`

Use `:hook` when a package should load when a mode or event happens.

```elisp
(use-package visual-fill-column
  :hook (org-mode . visual-fill-column-mode))
```

This means `visual-fill-column` does not need to load during startup. It can load when `org-mode` runs.

### `:bind`

Use `:bind` when a package should load when you press a key.

```elisp
(use-package magit
  :bind ("C-x g" . magit-status))
```

The package can stay unloaded until you invoke the binding.

### `:commands`

Use `:commands` when a package should load when one of its interactive commands is called.

```elisp
(use-package helpful
  :commands (helpful-callable
             helpful-variable
             helpful-command
             helpful-key))
```

This is useful for packages you only use on demand.

### `:mode`

Use `:mode` for major modes tied to file extensions.

```elisp
(use-package typescript-mode
  :mode "\\.ts\\'"
  :hook (typescript-mode . lsp-deferred))
```

Now `typescript-mode` loads when you open a TypeScript file, not when Emacs starts.

### `:after`

Use `:after` when one package only makes sense after another package loads.

```elisp
(use-package evil-collection
  :after evil
  :config
  (evil-collection-init))
```

You can also wait for multiple packages:

```elisp
(use-package some-package
  :after (evil general))
```

`use-package` also supports more advanced forms, such as loading after any one of several packages, but the simple list form covers many cases.

### `:defer`

Use `:defer` as the catch-all when none of the more specific options apply.

```elisp
(use-package hydra
  :defer t)
```

You can also defer for a number of idle seconds after startup:

```elisp
(use-package which-key
  :defer 1
  :config
  (which-key-mode))
```

That means Emacs can finish starting first, then load the package shortly afterward while idle.

### `:demand`

Use `:demand t` when a package must load immediately even though other keywords would normally defer it.

```elisp
(use-package evil
  :demand t
  :config
  (evil-mode 1))
```

You usually need this only for core packages that your configuration depends on right away.

## Use `:init` and `:config` Carefully

In `use-package`, `:init` runs before the package is loaded. `:config` runs after the package is loaded.

That distinction matters for startup performance.

If you enable a mode in `:init`, you may accidentally cause early loading or early work. For example:

```elisp
(use-package which-key
  :defer 1
  :init
  (which-key-mode))
```

That defeats the point of deferring `which-key`, because the mode is enabled during initialization.

Prefer:

```elisp
(use-package which-key
  :defer 1
  :config
  (which-key-mode))
```

Now `which-key-mode` starts when the package actually loads.

## Diagnose What Loads Early

While tuning, it helps to see what loads during startup.

You can temporarily add messages inside a package's `:config` block:

```elisp
(use-package org
  :config
  (message "org loaded"))
```

A better global diagnostic is:

```elisp
(setq use-package-verbose t)
```

This makes `use-package` report package loading and configuration timing in the `*Messages*` buffer.

Use it temporarily. It is useful while tuning, but noisy during normal use.

## Defer Org Correctly

Large packages like Org can have a visible impact on startup time.

A `:hook` may not be enough if some top-level code loads Org indirectly.

For example, this runs immediately and can cause Org to load during startup:

```elisp
(org-babel-do-load-languages
 'org-babel-load-languages
 '((emacs-lisp . t)
   (python . t)))
```

Wrap it so it runs only after Org loads:

```elisp
(with-eval-after-load 'org
  (org-babel-do-load-languages
   'org-babel-load-languages
   '((emacs-lisp . t)
     (python . t))))
```

The same idea applies to Org Tempo:

```elisp
(with-eval-after-load 'org
  (require 'org-tempo)
  (add-to-list 'org-structure-template-alist '("sh" . "src shell"))
  (add-to-list 'org-structure-template-alist '("el" . "src emacs-lisp"))
  (add-to-list 'org-structure-template-alist '("py" . "src python")))
```

If you use `org-capture` or `org-agenda`, list those commands so they can load Org when called:

```elisp
(use-package org
  :commands (org-capture org-agenda)
  :hook (org-mode . efs/org-mode-setup))
```

This keeps Org available when you need it without forcing it into every startup.

## Defer LSP and Debugging Tools

LSP and debugging packages are useful, but they do not need to load until you open a programming buffer or start a debug session.

For `lsp-mode`:

```elisp
(use-package lsp-mode
  :commands (lsp lsp-deferred)
  :hook (lsp-mode . efs/lsp-mode-setup))
```

For UI extras:

```elisp
(use-package lsp-ui
  :hook (lsp-mode . lsp-ui-mode))

(use-package lsp-treemacs
  :after lsp)

(use-package lsp-ivy
  :after lsp)
```

For DAP mode, avoid loading it until you actually debug:

```elisp
(use-package dap-mode
  :commands dap-debug)
```

This is often a meaningful win because LSP-related packages can pull in a lot of code.

## Defer Language Modes

Language modes should generally load when you open files in that language.

```elisp
(use-package typescript-mode
  :mode "\\.ts\\'"
  :hook (typescript-mode . lsp-deferred)
  :custom
  (typescript-indent-level 2))
```

For built-in modes, hooks are usually enough:

```elisp
(use-package python
  :ensure nil
  :hook (python-mode . lsp-deferred))
```

If you use helper packages for a language, load them after the language mode:

```elisp
(use-package pyvenv
  :after python)
```

## Defer Git Packages

Magit is excellent, but it is not needed until you ask for Git status.

```elisp
(use-package magit
  :commands magit-status)
```

If you use Forge with Magit:

```elisp
(use-package forge
  :after magit)
```

This can save a noticeable amount of startup time because Magit and its dependencies are substantial.

## Defer Shell and File Packages

Terminal, shell, and Dired extensions usually do not need to load at startup.

```elisp
(use-package term
  :ensure nil
  :commands term)

(use-package vterm
  :commands vterm)

(use-package eshell-git-prompt
  :after eshell)
```

Dired itself is built in, but extensions can wait:

```elisp
(use-package dired
  :ensure nil
  :commands (dired dired-jump))

(use-package dired-single
  :commands (dired-single-buffer dired-single-up-directory))

(use-package dired-open
  :commands dired-open-file)

(use-package all-the-icons-dired
  :hook (dired-mode . all-the-icons-dired-mode))
```

The pattern is the same: if a package supports a command, hook, mode, or parent package, use that as the loading trigger.

## Watch for Top-Level Elisp

Not all startup cost comes from `use-package` forms.

Loose top-level code can load packages early:

```elisp
(require 'org-tempo)
(org-babel-do-load-languages ...)
```

If that code belongs to a package, wrap it:

```elisp
(with-eval-after-load 'org
  ...)
```

This is one of the most common reasons a package loads even though the `use-package` block appears to be deferred.

## Tune Garbage Collection During Startup

Emacs Lisp is garbage collected. During startup, loading packages allocates memory, and Emacs may pause repeatedly to clean up.

You can reduce those pauses by raising `gc-cons-threshold` during startup.

Near the top of your configuration:

```elisp
(setq gc-cons-threshold (* 50 1000 1000))
```

This temporarily allows much more allocation before garbage collection runs.

After startup, lower it again:

```elisp
(setq gc-cons-threshold (* 2 1000 1000))
```

In the transcript, raising the threshold reduced startup garbage collections from about 23 to about 4 or 5 and shaved off roughly another second.

Do not leave the threshold extremely high forever. That can make Emacs use much more memory over time. A moderately higher runtime value, such as 2 MB, can still feel better than the very low default.

## Consider `gcmh`

Another option is the `gcmh` package, short for Garbage Collector Magic Hack.

It adjusts garbage collection behavior while Emacs is running. The basic setup is:

```elisp
(use-package gcmh
  :config
  (gcmh-mode 1))
```

The idea is to raise the garbage collection threshold while commands are running and lower it when Emacs is idle, so cleanup happens at less disruptive times.

You can use this as an alternative to manual GC tuning, or experiment with both approaches and measure which feels better in your configuration.

## Defer Everything by Default

`use-package` also has a global option:

```elisp
(setq use-package-always-defer t)
```

This makes packages deferred by default.

That can be cleaner than manually adding `:defer` everywhere, but it changes the burden: now you must explicitly mark the packages that must load immediately.

For example:

```elisp
(use-package general
  :demand t
  :config
  ...)

(use-package evil
  :demand t
  :config
  (evil-mode 1))
```

If you turn on `use-package-always-defer` without adding `:demand t` to your core packages, your configuration may break because keybinding helpers or foundational modes never load when your startup code expects them.

This can be a good strategy, but switch to it deliberately and test carefully.

## A Practical Optimization Process

Use this order:

1. Add startup timing.
2. Enable `use-package-verbose` temporarily.
3. Restart Emacs and inspect `*Messages*`.
4. Find large packages loading during startup.
5. Add `:commands`, `:hook`, `:mode`, `:after`, or `:defer`.
6. Wrap loose package-specific code in `with-eval-after-load`.
7. Restart and measure again.
8. Tune `gc-cons-threshold`.
9. Remove temporary diagnostic messages and `use-package-verbose`.

Do not change everything at once. Deferring packages can subtly change load order, and load order bugs are easier to solve when you make one small change at a time.

## Example Startup Performance Block

A compact startup performance section can look like this:

```elisp
;; Temporarily raise the threshold during startup.
(setq gc-cons-threshold (* 50 1000 1000))

(defun efs/display-startup-time ()
  (message "Emacs loaded in %s with %d garbage collections."
           (format "%.2f seconds"
                   (float-time
                    (time-subtract after-init-time before-init-time)))
           gcs-done))

(add-hook 'emacs-startup-hook #'efs/display-startup-time)

;; Lower the threshold again after startup.
(add-hook 'emacs-startup-hook
          (lambda ()
            (setq gc-cons-threshold (* 2 1000 1000))))
```

This gives you measurement and a simple GC improvement without changing the rest of your configuration.

## What Counts as Success?

In the video, the Emacs From Scratch configuration started around 5 seconds in a virtual machine while streaming. After deferring large packages and tuning garbage collection, it dropped to around 2.7 seconds.

The exact numbers will differ on your machine.

The important outcome is not a particular benchmark. It is that your startup work becomes intentional:

- core UI loads immediately
- expensive tools wait until needed
- package-specific setup runs after the package loads
- garbage collection does less work during startup

That makes Emacs start faster and usually makes day-to-day use feel smoother too.
