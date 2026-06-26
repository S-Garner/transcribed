# Emacs From Scratch 8: LSP Mode and IDE Features

The previous episodes built a strong Emacs foundation: UI polish, keybindings, project navigation, Git workflows, Org Mode, and literate configuration. This episode turns Emacs into a more capable programming environment with `lsp-mode`.

`lsp-mode` connects Emacs to language servers. Language servers provide editor features that people expect from modern IDEs: completion, jump to definition, references, rename refactoring, diagnostics, code actions, formatting, and more.

By the end, you will know how to:

- understand what the Language Server Protocol is
- install and configure `lsp-mode`
- enable LSP for a language mode
- install a language server
- use completion, signature help, references, definitions, and rename
- inspect diagnostics and run code actions
- add better completion UI with Company
- add richer LSP UI with `lsp-ui`
- add project and symbol views with Treemacs
- search workspace symbols with Ivy
- add a practical commenting package

## What LSP Is

LSP stands for Language Server Protocol.

The idea is simple: instead of every editor implementing deep language support separately, an editor can talk to an external language server. The language server understands a programming language and answers questions from the editor.

For example, Emacs can ask:

- What completions are available here?
- What is the type of this symbol?
- Where is this function defined?
- Where is this class referenced?
- Is there a syntax or type error in this file?
- Can this symbol be renamed everywhere safely?
- Are there code actions that fix this warning?

The language server sends structured responses back to the editor.

This matters because Emacs does not need a custom, hand-written integration for every feature of every language. It can use the same protocol that other editors use.

## Install `lsp-mode`

Add this to your Emacs config:

```elisp
(use-package lsp-mode
  :commands (lsp lsp-deferred)
  :init
  (setq lsp-keymap-prefix "C-c l")
  :config
  (lsp-enable-which-key-integration t))
```

Here is what the configuration does:

- `:commands (lsp lsp-deferred)` autoloads `lsp-mode` when either command is used.
- `lsp-keymap-prefix` sets the prefix for LSP commands.
- `C-c l` becomes the key prefix for language-server actions.
- `lsp-enable-which-key-integration` makes Which Key show descriptions for LSP commands.

The prefix matters because `lsp-mode` has many commands. A dedicated prefix gives you one predictable place to find them.

## Use `lsp` vs `lsp-deferred`

`lsp` starts the language server immediately.

`lsp-deferred` waits until the buffer is actually used before starting the server. This is useful with completion or buffer-switching tools because previewing buffers should not necessarily start language servers.

For hooks, prefer:

```elisp
lsp-deferred
```

That gives you a less surprising startup experience.

## Add a Language Mode

`lsp-mode` works inside language-specific major modes. Some modes are built into Emacs, but many languages need a package.

For TypeScript, install `typescript-mode`:

```elisp
(use-package typescript-mode
  :mode "\\.ts\\'"
  :hook (typescript-mode . lsp-deferred)
  :custom
  (typescript-indent-level 2))
```

This does three things:

- opens `.ts` files in `typescript-mode`
- starts LSP when a TypeScript buffer is opened
- sets TypeScript indentation to 2 spaces

For other languages, the same pattern applies:

1. Install or enable the major mode.
2. Add an LSP hook for that mode.
3. Install the matching language server.

## Install a Language Server

`lsp-mode` is only the Emacs client. You also need a language server for the language you want to edit.

For TypeScript and JavaScript, one option is the TypeScript language server:

```bash
npm install -g typescript-language-server typescript
```

Different languages use different servers:

- TypeScript/JavaScript often use `typescript-language-server`.
- Python may use `pyright`, `pylsp`, or another server.
- C and C++ may use `clangd` or `ccls`.
- Rust commonly uses `rust-analyzer`.
- Go uses `gopls`.

Always check the `lsp-mode` documentation for your language. The Emacs package and the external language server are separate pieces.

## Open a File and Start LSP

Open a TypeScript file:

```text
C-x C-f index.ts
```

If `typescript-mode` is configured with the `lsp-deferred` hook and the language server is installed, `lsp-mode` starts automatically.

If it does not, run:

```text
M-x lsp
```

The first time you open a project, `lsp-mode` may ask for the project root. Choose the root directory of your project.

## Complete at Point

Emacs has a built-in command called:

```text
M-x completion-at-point
```

When LSP is active, `completion-at-point` can ask the language server for completions.

For example, if you type:

```typescript
console.
```

then run `completion-at-point`, the language server can offer methods and properties on `console`.

This is the most basic completion workflow. It proves the language server is connected and returning language-aware results.

## Get Signature Help

When you call a function, `lsp-mode` can show its signature in the minibuffer.

For example:

```typescript
console.log(
```

The minibuffer can show something like:

```text
log(message?: any, ...optionalParams: any[]): void
```

If a function has multiple overloads, you can cycle through them with:

```text
M-n
M-p
```

This is useful when you know the function you want but do not remember its parameters.

## Inspect Symbols

Move point over a symbol. `lsp-mode` can show type or documentation information in the minibuffer.

For example, on a TypeScript variable, it may show the inferred type. On a function, it may show the function signature.

The language server is doing the language analysis. Emacs is displaying the result.

## Find References

To find references of the symbol at point:

```text
C-c l g r
```

That is:

- `C-c l` for the LSP prefix
- `g` for go to
- `r` for references

This shows every place the symbol is used in the project, across files.

This matters because real projects are multi-file. Text search can find matching strings, but LSP references are semantic. The language server knows which symbol you mean.

## Jump to Definition

To jump to the definition of the symbol at point:

```text
C-c l g g
```

This asks the language server where the current symbol is defined and jumps there.

With Evil mode, you can often jump back with:

```text
C-o
```

Jump to definition is one of the highest-value IDE features. It lets you navigate code by meaning instead of by file path.

## Rename a Symbol

To rename the symbol at point:

```text
C-c l r r
```

That opens the rename command. Enter the new name, and the language server updates all relevant references.

This is much safer than search-and-replace because the language server understands scope and symbol identity.

For example, renaming a method updates calls to that method without changing unrelated text that happens to have the same characters.

## View Diagnostics

Diagnostics are errors, warnings, or suggestions reported by the language server.

Emacs includes `flymake`, and `lsp-mode` can feed diagnostics into it.

Open the diagnostics buffer:

```text
M-x flymake-show-buffer-diagnostics
```

or:

```text
M-x flymake-show-project-diagnostics
```

Diagnostics can include things like:

- syntax errors
- missing imports
- unused variables
- type errors
- unreachable code

This gives you feedback before running a compiler or test suite.

## Run Code Actions

When the language server knows how to fix or improve something, it can provide a code action.

If `lsp-mode` shows a lightbulb in the mode line or a diagnostic has an available action, run:

```text
C-c l a a
```

This opens or applies code actions for the current location.

Examples might include:

- remove an unused declaration
- add a missing import
- apply a quick fix
- organize imports
- convert syntax

Available code actions depend entirely on the language server.

## Format a Buffer

To ask the language server to format the current buffer:

```text
C-c l = =
```

or run:

```text
M-x lsp-format-buffer
```

Formatting support varies by language server. Some servers format well. Others need project-specific configuration. For some languages, a dedicated formatter package may still be better.

Use LSP formatting if it matches your project's conventions. Otherwise, wire in the formatter your project already uses.

## Add Company Mode for Completion UI

The built-in completion UI works, but Company gives a more familiar popup completion experience.

Add:

```elisp
(use-package company
  :after lsp-mode
  :hook (lsp-mode . company-mode)
  :bind
  (:map company-active-map
        ("<tab>" . company-complete-selection)
        :map lsp-mode-map
        ("<tab>" . company-indent-or-complete-common))
  :custom
  (company-minimum-prefix-length 1)
  (company-idle-delay 0.0))
```

This configuration:

- enables Company when LSP starts
- makes `TAB` accept the selected completion
- asks for completions after one character
- shows completions immediately

This makes completion feel much more like modern programming editors.

## Add Company Box

`company-box` improves the appearance of Company completions.

Add:

```elisp
(use-package company-box
  :hook (company-mode . company-box-mode))
```

This can add a nicer popup and icons for completion candidates, depending on your font/icon setup.

It is mostly visual polish, but it can make completions easier to scan.

## Add `lsp-ui`

`lsp-ui` adds optional UI enhancements for `lsp-mode`.

Add:

```elisp
(use-package lsp-ui
  :hook (lsp-mode . lsp-ui-mode)
  :custom
  (lsp-ui-doc-position 'bottom))
```

Some useful features include:

- inline or sideline diagnostics
- documentation popups
- peek references
- extra actions and visual hints

These features can be helpful, but they can also become visually noisy. Treat them as optional. Turn on the pieces that help and turn off the ones that distract.

## Use Sideline Information

`lsp-ui-sideline` can show information on the right side of the buffer, such as diagnostics or available actions.

Toggle it with:

```text
C-c l T s
```

The exact Which Key labels may vary, but the LSP toggle menu is under:

```text
C-c l T
```

Sideline information can be useful in wide buffers. In narrow buffers, it may crowd the code. If it feels noisy, disable or reduce it.

## Use Documentation Popups

With `lsp-ui-doc`, moving point over a symbol can show documentation in a popup.

The configuration above places it at the bottom:

```elisp
(lsp-ui-doc-position 'bottom)
```

To focus the documentation frame:

```text
M-x lsp-ui-doc-focus-frame
```

To unfocus:

```text
M-x lsp-ui-doc-unfocus-frame
```

This is useful when documentation includes examples or longer explanations that you want to scroll or copy.

## Peek References

`lsp-ui` can show references inline in the current buffer instead of opening a separate results buffer.

Run:

```text
M-x lsp-ui-peek-find-references
```

This opens a temporary references view. You can move through references without fully switching context.

This is convenient for quick checks. For deeper investigation, `lsp-find-references` may be better because it gives you a persistent results buffer.

## Add Breadcrumbs

Breadcrumbs show where you are in the project and symbol hierarchy.

Add this setup function:

```elisp
(defun efs/lsp-mode-setup ()
  (setq lsp-headerline-breadcrumb-segments '(path-up-to-project file symbols))
  (lsp-headerline-breadcrumb-mode))
```

Then add it to `lsp-mode`:

```elisp
(use-package lsp-mode
  :commands (lsp lsp-deferred)
  :hook (lsp-mode . efs/lsp-mode-setup)
  :init
  (setq lsp-keymap-prefix "C-c l")
  :config
  (lsp-enable-which-key-integration t))
```

The breadcrumb line can show:

- project-relative path
- current file
- current class/function/method

This gives you orientation in large files and nested code.

## Add Treemacs and LSP Treemacs

Treemacs gives Emacs a side tree view. `lsp-treemacs` adds LSP-powered views such as symbols and references.

Add:

```elisp
(use-package treemacs)

(use-package lsp-treemacs
  :after lsp)
```

Open a file tree:

```text
M-x treemacs
```

Open a symbol tree for the current file:

```text
M-x lsp-treemacs-symbols
```

Open references in a tree:

```text
M-x lsp-treemacs-references
```

This creates an IDE-like side panel. It is especially helpful if you like seeing a file tree or symbol outline next to your code.

## Disable Line Numbers in Treemacs

If you enabled global line numbers earlier, Treemacs may show line numbers too. That is usually not useful.

Add `treemacs-mode-hook` to the list of modes where line numbers are disabled:

```elisp
(dolist (mode '(term-mode-hook
                shell-mode-hook
                eshell-mode-hook
                treemacs-mode-hook))
  (add-hook mode (lambda () (display-line-numbers-mode 0))))
```

This keeps utility buffers cleaner.

## Add Ivy LSP

`lsp-ivy` adds Ivy-based LSP search commands.

Add:

```elisp
(use-package lsp-ivy)
```

Search workspace symbols:

```text
M-x lsp-ivy-workspace-symbol
```

Type part of a class, function, or symbol name, then select the result to jump there.

This is useful when you know the name of something but not the file it lives in.

## LSP Works Beyond TypeScript

The same pattern works for other languages:

1. Install a major mode.
2. Install the language server.
3. Add an LSP hook.
4. Open a project file.

For C or C++, you might use `ccls` or `clangd`.

Example:

```elisp
(use-package ccls
  :hook ((c-mode c++-mode objc-mode cuda-mode) . lsp-deferred))
```

The language server needs to understand your project. Some languages have clear project files. C and C++ can require extra setup depending on build system complexity.

## Emacs Lisp Usually Does Not Need LSP

Emacs Lisp already has strong built-in integration in Emacs. You can use commands like:

```text
M-.
M-?
```

or Xref commands to jump to definitions and references.

Because Emacs itself understands Emacs Lisp well, you usually do not need a separate language server for it.

## Add Better Commenting

Emacs has built-in commenting commands, but the behavior may not always match what you expect from modern editors.

`evil-nerd-commenter` provides convenient commenting commands. It works even if you are not using Evil as your main editing style.

Add:

```elisp
(use-package evil-nerd-commenter
  :bind ("M-/" . evilnc-comment-or-uncomment-lines))
```

Now:

- select lines and press `M-/` to comment or uncomment them
- press `M-/` on a single line to toggle that line
- use the same binding across many languages

The package chooses the appropriate comment syntax for the current mode.

## Full Example Configuration

Here is a combined version of the LSP-related configuration from this tutorial:

```elisp
(defun efs/lsp-mode-setup ()
  (setq lsp-headerline-breadcrumb-segments '(path-up-to-project file symbols))
  (lsp-headerline-breadcrumb-mode))

(use-package lsp-mode
  :commands (lsp lsp-deferred)
  :hook (lsp-mode . efs/lsp-mode-setup)
  :init
  (setq lsp-keymap-prefix "C-c l")
  :config
  (lsp-enable-which-key-integration t))

(use-package typescript-mode
  :mode "\\.ts\\'"
  :hook (typescript-mode . lsp-deferred)
  :custom
  (typescript-indent-level 2))

(use-package company
  :after lsp-mode
  :hook (lsp-mode . company-mode)
  :bind
  (:map company-active-map
        ("<tab>" . company-complete-selection)
        :map lsp-mode-map
        ("<tab>" . company-indent-or-complete-common))
  :custom
  (company-minimum-prefix-length 1)
  (company-idle-delay 0.0))

(use-package company-box
  :hook (company-mode . company-box-mode))

(use-package lsp-ui
  :hook (lsp-mode . lsp-ui-mode)
  :custom
  (lsp-ui-doc-position 'bottom))

(use-package treemacs)

(use-package lsp-treemacs
  :after lsp)

(use-package lsp-ivy)

(use-package evil-nerd-commenter
  :bind ("M-/" . evilnc-comment-or-uncomment-lines))
```

Install the TypeScript language server separately:

```bash
npm install -g typescript-language-server typescript
```

## What This Gives You

With `lsp-mode`, Emacs can provide many of the programming features people expect from IDEs: semantic completion, signature help, jump to definition, find references, rename refactoring, diagnostics, code actions, formatting, symbol search, and project outlines.

The important model is that Emacs is the client. The language server provides language intelligence. Your Emacs configuration decides how much UI you want around that intelligence.

Start with plain `lsp-mode`, add Company for completion, then add `lsp-ui`, Treemacs, and Ivy integrations only where they improve your workflow.
