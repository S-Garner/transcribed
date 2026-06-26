# Emacs From Scratch 9: Terminals and Shells

As you get more comfortable in Emacs, it is natural to move more of your workflow into it. For programming, that usually means you need a terminal or shell for running commands, compiling code, checking Git state, starting servers, and using command-line tools.

Emacs gives you several options, and they are not all the same:

- `term`
- `ansi-term`
- `vterm`
- `shell`
- `eshell`

Each one has different tradeoffs. Some emulate a real terminal. Some integrate more deeply with Emacs. Some work better on Windows. Some are faster. Some are more configurable.

This tutorial explains what each option is for, how to configure it, and when you might choose one over another.

## Terminal Emulator vs Shell

Before comparing modes, it helps to separate two ideas.

A shell is the program that reads commands and runs them. Examples include:

- `bash`
- `zsh`
- `fish`
- `powershell.exe`
- `cmd.exe`
- `eshell`

A terminal emulator is the program that displays terminal output and sends keyboard input to the shell. It interprets control codes for cursor movement, colors, screen updates, and full-screen terminal programs.

This distinction matters because not every Emacs shell mode is a terminal emulator.

For example:

- `term`, `ansi-term`, and `vterm` are terminal emulators.
- `shell` runs a shell process but is not a full terminal emulator.
- `eshell` is a shell written in Emacs Lisp.

If a program expects a real terminal, such as `htop`, `vim`, or some interactive REPLs, it may not work correctly in `shell` or `eshell`.

## Use `term`

`term` is a terminal emulator built into Emacs. It is written in Emacs Lisp.

Run it with:

```text
M-x term
```

Emacs asks which program to run. By default, it usually uses your `SHELL` environment variable.

You can inspect that variable inside Emacs:

```text
M-: (getenv "SHELL")
```

In `term`, you can run normal terminal commands:

```bash
ls
git status
htop
```

The advantage of `term` is that it comes with Emacs and works out of the box on Unix-like systems.

The downside is performance. Because the terminal emulator is written in Emacs Lisp, very large output can be slower than a native terminal emulator.

## Configure Prompt Navigation in `term`

`term` can jump between prompts, but it needs to know what your prompt looks like.

Configure `term-prompt-regexp`:

```elisp
(use-package term
  :config
  (setq term-prompt-regexp "^[^#$%>\n]*[#$%>] *"))
```

This regular expression tries to match common shell prompts ending in characters like `$`, `#`, `%`, or `>`.

Then use:

```text
C-c C-p
C-c C-n
```

to jump to the previous or next prompt.

If you use a custom shell prompt, you may need to customize `term-prompt-regexp` so Emacs can recognize it.

## Understand Line Mode and Char Mode

`term` has two input modes.

Line mode lets Emacs handle many keybindings. This makes the buffer feel more like a normal Emacs buffer.

Char mode sends most keys directly to the terminal program. This is necessary for programs like `vim`, `htop`, or anything that expects raw terminal input.

Switch to char mode:

```text
C-c C-k
```

Switch back to line mode:

```text
C-c C-j
```

This matters because if a terminal program is not receiving your keys correctly, you may be in the wrong mode.

## Use `term` With Evil Mode

If you use Evil and Evil Collection, `term` becomes more comfortable.

In normal state, you can navigate and select text like a normal Evil buffer.

In insert state, Evil Collection can switch `term` into char mode so your keystrokes go to the terminal program.

This is useful, but there is a sharp edge: do not edit a command line in Evil normal state and expect the shell process to know about those edits.

For example, if you type:

```bash
git status
```

then leave insert state and delete `status` with Evil commands, the terminal process may not receive that edit. When you press Enter, the shell may still think you typed the original text.

The practical rule is:

- edit command input in insert/char mode
- use normal mode for navigating, selecting, and copying terminal output

Evil Collection also gives prompt movement bindings:

```text
[[
]]
```

These move to previous and next prompts.

## Add 256-Color Support to `term`

By default, `term` may not display extended colors correctly.

Install:

```elisp
(use-package eterm-256color
  :hook (term-mode . eterm-256color-mode))
```

The package may need to compile a terminal definition. On Linux, you may need tools from `ncurses`, especially `tic`.

On Debian or Ubuntu-like systems:

```bash
sudo apt install ncurses-bin
```

Once enabled, programs that use 256-color terminal output should display more accurately.

This is useful if you run tools with rich terminal colors or prompts.

## Use `ansi-term`

`ansi-term` is closely related to `term`. It is also built into Emacs and uses much of the same terminal emulation infrastructure.

Run it with:

```text
M-x ansi-term
```

The main practical difference is buffer behavior. `ansi-term` more readily creates separate terminal buffers, while `term` often reuses the existing `*terminal*` buffer unless you rename it.

If you use `term` and want another terminal, run:

```text
M-x rename-uniquely
```

Then run `M-x term` again.

For most users, `term` and `ansi-term` occupy the same category: built-in terminal emulators for Unix-like systems.

## Windows Note for `term` and `ansi-term`

`term` and `ansi-term` assume a Unix-like environment. They are not good choices on native Windows.

On Windows, prefer:

- `shell`
- `eshell`

If you use WSL, you may still need extra configuration depending on how Emacs is installed and how you want to access the WSL shell.

## Use `vterm`

`vterm` is an external package that provides a faster terminal emulator. Unlike `term`, it uses a native compiled module backed by `libvterm`.

Install:

```elisp
(use-package vterm
  :commands vterm
  :config
  (setq vterm-max-scrollback 10000))
```

Then run:

```text
M-x vterm
```

The `vterm-max-scrollback` setting controls how many lines of output are retained.

Higher values keep more history but can affect performance and memory. `10000` is a reasonable starting point.

## Install `vterm` Dependencies

Because `vterm` uses a native module, it needs build tools.

Common requirements include:

- Emacs built with module support
- C compiler
- CMake
- libtool
- make
- `libvterm`

On many Linux distributions, you can install the needed packages through your package manager.

If `vterm` fails to build, read the error carefully. It usually tells you which tool is missing.

This setup cost is the main downside of `vterm`. The upside is that it behaves much more like a real terminal and handles large output better.

## Improve `vterm` Prompt Detection

Like `term`, `vterm` can use prompt detection for prompt navigation. `term-prompt-regexp` may help:

```elisp
(setq term-prompt-regexp "^[^#$%>\n]*[#$%>] *")
```

For better results, `vterm` supports shell-side configuration. That means your shell prompt can print special markers that help `vterm` know exactly where prompts are.

If prompt navigation is unreliable with a custom prompt, check the `vterm` documentation for shell integration snippets.

## Why Choose `vterm`

Choose `vterm` if:

- you are on Linux or macOS
- you want the closest thing to a real terminal inside Emacs
- you run full-screen terminal applications
- you often handle large terminal output
- you are willing to install native dependencies

`vterm` is usually the best terminal-emulator experience in Emacs if your platform supports it.

## Use `shell`

`shell` runs an inferior shell process inside an Emacs buffer. It is not a full terminal emulator.

Run it with:

```text
M-x shell
```

You can run normal shell commands:

```bash
ls
git status
make
```

But full-screen terminal programs may not work:

```bash
htop
vim
```

This is because `shell` is not emulating a terminal. It is managing interaction with a shell process in a more Emacs-native way.

## Why Use `shell`

`shell` is a useful middle ground.

Compared to `term` or `vterm`, it integrates more naturally with Emacs text editing. For example, if you use Evil, editing the command line in normal state works more predictably because Emacs is managing the input.

It also works better on Windows than `term` or `vterm`.

Choose `shell` if:

- you want to run ordinary shell commands
- you want better Emacs editing behavior
- you do not need full terminal emulation
- you are on Windows and want to use PowerShell or `cmd.exe`

## Search Shell History With Counsel

If you use Counsel, you can search shell history:

```text
M-x counsel-shell-history
```

This gives you fuzzy search over previous commands.

You can also move through history with:

```text
M-p
M-n
```

This is one of the advantages of using shell-like modes inside Emacs: history becomes searchable with your existing completion tools.

## Configure Shell on Windows

On Windows, you can tell Emacs which shell to use with `explicit-shell-file-name`.

For PowerShell:

```elisp
(setq explicit-shell-file-name "powershell.exe")
(setq explicit-powershell.exe-args '())
```

For PowerShell Core:

```elisp
(setq explicit-shell-file-name "pwsh.exe")
(setq explicit-pwsh.exe-args '())
```

The `explicit-...-args` variable matters because Emacs constructs an argument variable name from the shell executable. If the wrong default arguments are passed, the shell may start with errors.

## Use `eshell`

`eshell` is a shell written in Emacs Lisp.

Run it with:

```text
M-x eshell
```

Unlike `shell`, `eshell` is not running Bash or Zsh as the shell. It implements many shell commands itself in Emacs Lisp.

That means commands like these work consistently across platforms:

```bash
ls
cd
rm
cp
mv
```

This is especially interesting on Windows because it gives you a Unix-like shell experience inside Emacs.

## Why `eshell` Is Different

Because `eshell` is written in Emacs Lisp, it can interact deeply with Emacs.

You can run Emacs Lisp expressions:

```elisp
(+ 200 50)
```

You can call Emacs commands:

```text
find-file init.el
```

You can send command output to an Emacs buffer:

```text
echo hello > #<buffer test-buffer>
```

This makes `eshell` more than a shell. It is also an Emacs Lisp REPL and command environment.

## Configure an Eshell Prompt

You can customize the Eshell prompt. One convenient package is `eshell-git-prompt`.

Add:

```elisp
(use-package eshell-git-prompt)
```

Then in Eshell, run:

```text
use-theme
```

You can choose from prompt themes such as:

- `robbyrussell`
- `git-radar`
- `powerline`

For example:

```text
use-theme powerline
```

This gives Eshell a richer prompt with Git information.

## Configure Eshell History and Buffer Behavior

Add:

```elisp
(defun efs/configure-eshell ()
  (add-hook 'eshell-pre-command-hook 'eshell-save-some-history)
  (add-hook 'eshell-output-filter-functions 'eshell-truncate-buffer)

  (setq eshell-history-size 10000
        eshell-buffer-maximum-lines 10000
        eshell-hist-ignoredups t
        eshell-scroll-to-bottom-on-input t))

(use-package eshell
  :hook (eshell-first-time-mode . efs/configure-eshell))
```

This does several useful things:

- saves history before commands run
- truncates long Eshell buffers
- keeps up to 10,000 history entries
- avoids duplicate history entries
- scrolls to the bottom when you type input

Saving history early is important because Eshell otherwise may not write history until the shell exits. If Emacs crashes, recent history could be lost.

## Add Eshell Keybindings

If you use Evil, you may want a few Eshell-specific bindings:

```elisp
(use-package eshell
  :hook (eshell-first-time-mode . efs/configure-eshell)
  :bind (:map eshell-mode-map
         ("C-r" . counsel-esh-history)
         ("<home>" . eshell-bol)))
```

`C-r` opens searchable Eshell history with Counsel.

`eshell-bol` moves to the beginning of the command input instead of the beginning of the whole visual line.

This makes Eshell feel more like a real shell while preserving Emacs editing behavior.

## Run Visual Commands From Eshell

Some programs need a real terminal interface. Eshell can hand those commands off to a terminal emulator.

Configure visual commands:

```elisp
(use-package eshell
  :config
  (with-eval-after-load 'esh-opt
    (setq eshell-destroy-buffer-when-process-dies t)
    (setq eshell-visual-commands '("bash" "fish" "htop" "ssh" "top" "zsh"))))
```

Now when you run:

```bash
htop
```

from Eshell, Emacs opens it in a terminal-style buffer instead of trying to run it directly inside Eshell.

This is important because Eshell is not a terminal emulator.

## Eshell Pros

Eshell has several strong advantages:

- works across Linux, macOS, and Windows
- provides many Unix-like commands in Emacs Lisp
- integrates deeply with Emacs buffers and commands
- can run Emacs Lisp directly
- can pipe output into Emacs buffers
- works with TRAMP for remote workflows
- is highly customizable

If you want an Emacs-native shell experience, Eshell is the most interesting option.

## Eshell Cons

Eshell also has rough edges:

- built-in completions are weaker than Bash or Fish
- Emacs Lisp implementations of commands can be slower
- piping is not always as powerful as in real shells
- subshell syntax differs from Bash
- interactive programs can behave strangely
- environment tools like `nvm` or `virtualenv` may need extra packages or configuration
- on Windows, performance can still be uneven

The practical takeaway: Eshell is powerful, but it is not a drop-in Bash replacement for every workflow.

## Recommendations

Use `term` or `ansi-term` if:

- you want a built-in terminal emulator
- you are on Linux or macOS
- you do not want extra dependencies

Use `vterm` if:

- you want the best terminal emulation inside Emacs
- you are comfortable installing native dependencies
- you run interactive or high-output terminal programs

Use `shell` if:

- you want a real shell process with better Emacs text integration
- you are on Windows
- you do not need full terminal emulation

Use `eshell` if:

- you want a cross-platform Emacs-native shell
- you like the idea of calling Emacs functions from a shell
- you want deep Emacs integration
- you can tolerate a few differences from Bash/Zsh

## Full Example Configuration

Here is a combined configuration from this tutorial:

```elisp
(use-package term
  :config
  (setq term-prompt-regexp "^[^#$%>\n]*[#$%>] *"))

(use-package eterm-256color
  :hook (term-mode . eterm-256color-mode))

(use-package vterm
  :commands vterm
  :config
  (setq vterm-max-scrollback 10000))

(defun efs/configure-eshell ()
  (add-hook 'eshell-pre-command-hook 'eshell-save-some-history)
  (add-hook 'eshell-output-filter-functions 'eshell-truncate-buffer)

  (setq eshell-history-size 10000
        eshell-buffer-maximum-lines 10000
        eshell-hist-ignoredups t
        eshell-scroll-to-bottom-on-input t))

(use-package eshell
  :hook (eshell-first-time-mode . efs/configure-eshell)
  :bind (:map eshell-mode-map
         ("C-r" . counsel-esh-history)
         ("<home>" . eshell-bol))
  :config
  (with-eval-after-load 'esh-opt
    (setq eshell-destroy-buffer-when-process-dies t)
    (setq eshell-visual-commands '("bash" "fish" "htop" "ssh" "top" "zsh"))))

(use-package eshell-git-prompt)
```

Optional Windows shell configuration:

```elisp
(setq explicit-shell-file-name "powershell.exe")
(setq explicit-powershell.exe-args '())
```

## What This Gives You

Emacs gives you several ways to work with terminals and shells, and the best choice depends on what you need.

For raw terminal compatibility, use `vterm` when possible. For built-in terminal emulation, use `term` or `ansi-term`. For a shell process with better Emacs editing behavior, use `shell`. For the most Emacs-native experience, use `eshell`.

The most useful approach is not to pick one forever. Use `vterm` for programs that need a real terminal, `eshell` for Emacs-integrated workflows, and `shell` when you need a simple process shell with normal Emacs editing.
