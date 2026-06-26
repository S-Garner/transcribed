# Emacs From Scratch 7: Org Babel and Literate Configuration

The previous Org Mode episodes covered writing, planning, agenda views, capture templates, and habits. This episode focuses on a different part of Org Mode: Org Babel.

Org Babel lets you place source code blocks inside Org files, execute those blocks, collect their results, and write them out to real files. That makes it useful for notebooks, reproducible documents, and literate configuration.

In this tutorial, you will use Org Babel to organize an Emacs configuration in an Org file and tangle it into `init.el`.

By the end, you will know how to:

- write source blocks in Org Mode
- execute source blocks
- enable Babel languages
- disable confirmation prompts when appropriate
- create source block shortcuts with structure templates
- tangle source blocks into files
- set file-wide tangle defaults
- automatically tangle your Emacs config on save
- generate other application config files from Org
- reuse values between blocks with noweb references

## What Org Babel Does

Org Babel gives Org Mode source blocks real behavior.

A source block looks like this:

```org
#+begin_src emacs-lisp
(+ 1 2)
#+end_src
```

Org Babel can:

- execute the block and insert results
- export the block and results into documents
- collect blocks and write them to external files
- let blocks pass values to each other

This is why Org Babel is useful for configuration. You can write prose explaining what a section does, keep the actual configuration in source blocks, and generate the real config files from those blocks.

## Create a Source Block

Open an Org file:

```text
C-x C-f emacs.org
```

Add a source block:

```org
#+begin_src emacs-lisp
(+ 40 2)
#+end_src
```

Place point inside the block and run:

```text
C-c C-c
```

Org evaluates the block and inserts a result:

```org
#+RESULTS:
: 42
```

This is the simplest Org Babel workflow: write code, evaluate it, see the result in the document.

## Configure Babel Languages

Org Babel does not execute every language by default. If you try to run a Python block before enabling Python support, Org may report that there is no execute function for Python.

Enable languages with:

```elisp
(org-babel-do-load-languages
 'org-babel-load-languages
 '((emacs-lisp . t)
   (python . t)))
```

Now Org knows how to evaluate Emacs Lisp and Python blocks.

For Python blocks, you still need Python installed and available in Emacs' environment. If Emacs cannot find `python`, the Babel language may be enabled but execution will still fail.

This distinction matters:

- Babel language support tells Org how to run a language.
- The external runtime still needs to exist on your system.

## Control Result Types

Source blocks can return different kinds of results.

This block returns a value:

```org
#+begin_src emacs-lisp :results value
(+ 40 2)
#+end_src
```

The result is the value of the expression:

```org
#+RESULTS:
: 42
```

This block returns output:

```org
#+begin_src emacs-lisp :results output
(message "hello")
#+end_src
```

The result is the printed or emitted output.

Use `value` when the final expression matters. Use `output` when printed text matters.

## Disable Evaluation Confirmation Carefully

By default, Org asks before evaluating code blocks. That is a security feature. Source blocks can run arbitrary code.

For files you trust, you can disable the prompt:

```elisp
(setq org-confirm-babel-evaluate nil)
```

This makes `C-c C-c` execute blocks without asking.

Only do this if you are comfortable with the risk. It is convenient for your own configuration files, but you should be cautious with Org files from other people.

## Add Structure Templates

Typing `#+begin_src` and `#+end_src` repeatedly gets old quickly. Org structure templates let you type a short abbreviation and expand it into a block.

First, require `org-tempo`:

```elisp
(require 'org-tempo)
```

Then add templates:

```elisp
(add-to-list 'org-structure-template-alist '("sh" . "src shell"))
(add-to-list 'org-structure-template-alist '("el" . "src emacs-lisp"))
(add-to-list 'org-structure-template-alist '("py" . "src python"))
```

Now, in an Org buffer, type:

```text
<el
```

Then press:

```text
TAB
```

Org expands it to:

```org
#+begin_src emacs-lisp

#+end_src
```

This matters because literate configuration usually contains many source blocks. Structure templates make writing those blocks much faster.

## Tangle a Source Block

Tangling means writing source blocks out to real files.

For example, create this block:

```org
* Basic UI Configuration

#+begin_src emacs-lisp :tangle init-new.el
(setq inhibit-startup-message t)

(scroll-bar-mode -1)
(tool-bar-mode -1)
(tooltip-mode -1)
(menu-bar-mode -1)

(setq visible-bell t)
#+end_src
```

Run:

```text
M-x org-babel-tangle
```

Org writes the block to:

```text
init-new.el
```

This is the core mechanism behind literate configuration. The Org file is where you write and organize the configuration. The tangled file is what Emacs or another program actually loads.

## Tangle Multiple Blocks Into One File

You can have several blocks that all tangle into the same file:

```org
* Basic UI Configuration

#+begin_src emacs-lisp :tangle init-new.el
(setq inhibit-startup-message t)
#+end_src

* Font Configuration

#+begin_src emacs-lisp :tangle init-new.el
(set-face-attribute 'default nil
                    :font "Fira Code Retina"
                    :height 140)
#+end_src
```

When you run:

```text
M-x org-babel-tangle
```

Org concatenates both blocks into `init-new.el` in document order.

This matters because you can split your configuration into readable sections without splitting the generated file.

## Set File-Wide Tangle Defaults

Adding `:tangle init-new.el` to every block is tedious. You can define defaults at the top of the Org file.

Add this near the top:

```org
#+title: Emacs From Scratch Configuration
#+property: header-args:emacs-lisp :tangle init-new.el
```

Now every Emacs Lisp source block tangles to `init-new.el` unless the block overrides that setting.

Example:

```org
* Basic UI Configuration

#+begin_src emacs-lisp
(setq inhibit-startup-message t)
#+end_src
```

If Org does not pick up the new property immediately, restart Org Mode in the buffer:

```text
M-x org-mode
```

Then run:

```text
M-x org-babel-tangle
```

This is useful because top-level header arguments keep individual blocks clean.

## Use `:mkdirp yes` for Generated Paths

If you tangle to a path whose directory does not exist, Org errors.

For example:

```org
#+begin_src conf :tangle .config/some-app/config
value=42
#+end_src
```

If `.config/some-app/` does not exist, tangling fails.

Add `:mkdirp yes`:

```org
#+begin_src conf :tangle .config/some-app/config :mkdirp yes
value=42
#+end_src
```

Or set it globally:

```org
#+property: header-args :mkdirp yes
```

For Emacs Lisp-specific defaults, combine it with the earlier property:

```org
#+property: header-args:emacs-lisp :tangle init.el :mkdirp yes
```

This tells Org to create missing directories before writing tangled files.

## Auto-Tangle on Save

If `emacs.org` generates `init.el`, forgetting to tangle after edits is annoying. You will restart Emacs and wonder why your changes did not load.

You can automatically tangle your config whenever you save `emacs.org`.

Define a helper that only tangles your configuration Org file:

```elisp
(defun efs/org-babel-tangle-config ()
  (when (string-equal (buffer-file-name)
                      (expand-file-name "~/projects/emacs-from-scratch/emacs.org"))
    (let ((org-confirm-babel-evaluate nil))
      (org-babel-tangle))))
```

This function checks whether the current file is your Emacs configuration Org file. If it is, it tangles the file after save.

The `let` binding disables Babel confirmation only for this tangle operation:

```elisp
(let ((org-confirm-babel-evaluate nil))
  (org-babel-tangle))
```

That avoids a prompt every time you save.

## Add a Buffer-Local Save Hook

Now attach that helper to `after-save-hook` for Org buffers:

```elisp
(add-hook 'org-mode-hook
          (lambda ()
            (add-hook 'after-save-hook #'efs/org-babel-tangle-config nil 'local)))
```

The final `'local` argument makes the save hook local to the current buffer.

This keeps the behavior scoped to Org buffers and avoids accidentally adding a global after-save hook from every Org buffer you open.

## Move `init.el` Into `emacs.org`

To convert an existing `init.el` into a literate config:

1. Create `emacs.org`.
2. Add a title and header args.
3. Create sections that match the structure of your config.
4. Move related chunks of Emacs Lisp into source blocks.
5. Tangle to `init.el`.
6. Restart Emacs and verify the generated config loads.

At the top of `emacs.org`:

```org
#+title: Emacs From Scratch Configuration
#+property: header-args:emacs-lisp :tangle init.el :mkdirp yes
```

Then organize your config:

```org
* Package System Setup

#+begin_src emacs-lisp
(require 'package)

(setq package-archives
      '(("melpa" . "https://melpa.org/packages/")
        ("org" . "https://orgmode.org/elpa/")
        ("elpa" . "https://elpa.gnu.org/packages/")))

(package-initialize)
#+end_src

* Basic UI Configuration

#+begin_src emacs-lisp
(setq inhibit-startup-message t)
(scroll-bar-mode -1)
(tool-bar-mode -1)
(tooltip-mode -1)
(menu-bar-mode -1)
#+end_src

* Completion

#+begin_src emacs-lisp
(use-package ivy
  :config
  (ivy-mode 1))
#+end_src
```

When you tangle, Org writes one `init.el` containing the blocks in order.

This matters because order still matters. Package setup must happen before package configuration. Keybindings that depend on packages should happen after those packages are loaded.

## Use Org for Other Config Files

Org Babel is not limited to Emacs Lisp.

You can generate configuration for other programs too:

```org
* Applications

** Some App

#+begin_src conf :tangle .config/some-app/config :mkdirp yes
value=42
theme=dark
#+end_src
```

Running:

```text
M-x org-babel-tangle
```

writes:

```text
.config/some-app/config
```

This is the same idea as literate Emacs configuration, but applied to dotfiles. The Org file explains the configuration, and tangling produces the actual files that programs read.

## Map Source Block Languages to Modes

Org uses language names in source blocks for syntax highlighting. Sometimes you may want to map a source block language to a specific Emacs mode.

For example:

```elisp
(add-to-list 'org-src-lang-modes '("conf" . conf-unix))
```

Then a block like this:

```org
#+begin_src conf
value=42
#+end_src
```

uses `conf-unix-mode` style highlighting.

This is useful when a config format is not a programming language but still has a mode that can highlight it well.

## Reuse Values With Noweb References

Org Babel supports named blocks. You can refer to those blocks from other blocks using noweb syntax.

Create a named block:

```org
#+name: the-value
#+begin_src emacs-lisp
(+ 55 100)
#+end_src
```

Then use it in another block:

```org
#+begin_src conf :tangle .config/some-app/config :mkdirp yes :noweb yes
value=<<the-value()>>
#+end_src
```

After tangling, the generated config contains:

```conf
value=155
```

The `()` in `<<the-value()>>` tells Babel to evaluate the named block and use its result.

This matters because configuration can be computed. You can derive values from system-specific settings, environment details, shared constants, or other blocks.

## Use Per-System Settings

Noweb references become powerful when one Org configuration is shared across multiple machines.

For example:

```org
#+name: font-size
#+begin_src emacs-lisp
(if (string-equal system-name "laptop")
    140
  110)
#+end_src

#+begin_src emacs-lisp :noweb yes
(set-face-attribute 'default nil
                    :font "Fira Code Retina"
                    :height <<font-size()>>)
#+end_src
```

When tangled on your laptop, the generated config uses one value. On another machine, it uses another.

This is useful for differences like:

- high-DPI vs normal displays
- different font sizes
- different executable paths
- machine-specific package lists
- desktop vs laptop settings

## Check In Generated Files or Not?

If your Org file generates `init.el` and other dotfiles, you need to decide whether to check the generated files into Git.

Checking them in has advantages:

- you can see diffs of generated output
- startup works even before tangling
- changes are easier to inspect

Not checking them in also has advantages:

- the Org files are the only source of truth
- generated files never become stale in Git
- machine-specific generated files do not clutter commits

The practical recommendation is:

- Check generated files in if your setup is simple or you want easier debugging.
- Skip generated files only if you have reliable automation that regenerates them before use.

For a first literate config, checking in `init.el` is reasonable.

## Full Example

Here is a small `emacs.org` example:

```org
#+title: Emacs From Scratch Configuration
#+property: header-args:emacs-lisp :tangle init.el :mkdirp yes

* Package System Setup

#+begin_src emacs-lisp
(require 'package)

(setq package-archives
      '(("melpa" . "https://melpa.org/packages/")
        ("org" . "https://orgmode.org/elpa/")
        ("elpa" . "https://elpa.gnu.org/packages/")))

(package-initialize)

(unless package-archive-contents
  (package-refresh-contents))

(unless (package-installed-p 'use-package)
  (package-install 'use-package))

(require 'use-package)
(setq use-package-always-ensure t)
#+end_src

* Basic UI Configuration

#+begin_src emacs-lisp
(setq inhibit-startup-message t)
(scroll-bar-mode -1)
(tool-bar-mode -1)
(tooltip-mode -1)
(menu-bar-mode -1)
(setq visible-bell t)
#+end_src

* Org Babel Setup

#+begin_src emacs-lisp
(org-babel-do-load-languages
 'org-babel-load-languages
 '((emacs-lisp . t)
   (python . t)))

(setq org-confirm-babel-evaluate nil)

(require 'org-tempo)
(add-to-list 'org-structure-template-alist '("sh" . "src shell"))
(add-to-list 'org-structure-template-alist '("el" . "src emacs-lisp"))
(add-to-list 'org-structure-template-alist '("py" . "src python"))
#+end_src

* Auto-Tangle

#+begin_src emacs-lisp
(defun efs/org-babel-tangle-config ()
  (when (string-equal (buffer-file-name)
                      (expand-file-name "~/projects/emacs-from-scratch/emacs.org"))
    (let ((org-confirm-babel-evaluate nil))
      (org-babel-tangle))))

(add-hook 'org-mode-hook
          (lambda ()
            (add-hook 'after-save-hook #'efs/org-babel-tangle-config nil 'local)))
#+end_src
```

Run:

```text
M-x org-babel-tangle
```

This generates:

```text
init.el
```

Then restart Emacs and verify the generated `init.el` loads correctly.

## What This Gives You

Org Babel turns configuration into a document. You can group related settings under headings, collapse sections, explain why settings exist, and still generate ordinary config files that Emacs and other programs can load.

The main tradeoff is complexity. A literate configuration has one more generation step: `emacs.org` must be tangled into `init.el`. Auto-tangling on save removes most of that friction, but you still need to remember that the Org file is the source of truth.

For a growing Emacs config, this structure can make the system easier to understand and maintain.
