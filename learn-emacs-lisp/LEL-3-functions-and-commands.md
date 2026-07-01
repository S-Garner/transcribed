# Learn Emacs Lisp 3: Functions and Commands

Functions are where Emacs Lisp starts to become practical.

You can get far in your Emacs configuration by setting variables and calling functions that other people wrote. Eventually, though, you will want reusable behavior of your own: a hook function, a helper, a command, or a small package.

This tutorial covers:

- defining functions with `defun`
- required, optional, and rest arguments
- documentation strings
- anonymous functions with `lambda`
- calling functions with `funcall` and `apply`
- defining interactive commands
- prompting users for arguments
- starting a small dotfiles-management project

## What Is a Function?

A function is a reusable piece of code.

It can:

- accept input values as parameters
- perform work
- return a result
- be called by other code
- sometimes be called directly by users

In Emacs Lisp, most expressions return values. A function returns the value of the last expression in its body unless something unusual happens, such as an error.

Functions usually have names, but they do not have to. Anonymous functions are useful when you need a function value temporarily.

## Define a Basic Function

Use `defun` to define a function:

```elisp
(defun do-some-math (x y)
  (* (+ x 20)
     (- y 10)))
```

This defines a function named `do-some-math`.

It takes two parameters:

- `x`
- `y`

The body computes:

```elisp
(* (+ x 20)
   (- y 10))
```

Because that is the last expression in the function, its value becomes the return value.

Call it like any other Emacs Lisp function:

```elisp
(do-some-math 100 50)
;; => 4800
```

## Required Arguments

The parameters in this version are required:

```elisp
(defun do-some-math (x y)
  (* (+ x 20)
     (- y 10)))
```

Calling it with too few or too many arguments raises an error:

```elisp
(do-some-math 100)
;; error

(do-some-math 100 50 20)
;; error
```

Emacs Lisp is dynamically typed, so the function definition does not say what type `x` and `y` must be. If you pass bad values, an operation inside the function will usually complain later.

For small personal configuration functions, that is often fine. For package code, you may want clearer validation.

## Optional Arguments

Use `&optional` for arguments that may be omitted.

```elisp
(defun multiply-maybe (x &optional y z)
  (* x
     (or y 1)
     (or z 1)))
```

`x` is required.

`y` and `z` are optional.

If an optional argument is not provided, its value is `nil`.

The `or` expressions provide defaults:

```elisp
(or y 1)
```

means "use `y` if it is non-nil, otherwise use `1`."

Examples:

```elisp
(multiply-maybe 5)
;; => 5

(multiply-maybe 5 2)
;; => 10

(multiply-maybe 5 2 10)
;; => 100
```

If you want to skip an earlier optional argument but provide a later one, pass `nil`:

```elisp
(multiply-maybe 5 nil 10)
;; => 50
```

That uses the default for `y` and passes `10` for `z`.

## Rest Arguments

Use `&rest` when a function should accept any number of extra arguments.

```elisp
(defun multiply-many (x &rest operands)
  (dolist (operand operands)
    (when operand
      (setq x (* x operand))))
  x)
```

`x` is required.

Every argument after `x` is collected into the list `operands`.

Examples:

```elisp
(multiply-many 5)
;; => 5

(multiply-many 5 2)
;; => 10

(multiply-many 5 2 10)
;; => 100

(multiply-many 5 2 10 7)
;; => 700
```

This function skips `nil` operands:

```elisp
(multiply-many 5 nil 10)
;; => 50
```

Rest arguments are how functions like `+` can accept many values:

```elisp
(+ 2 3 4 6)
;; => 15
```

## Optional and Rest Together

You can combine `&optional` and `&rest`.

```elisp
(defun multiply-more (x &optional y &rest operands)
  (setq x (* x (or y 1)))
  (dolist (operand operands)
    (when operand
      (setq x (* x operand))))
  x)
```

Here:

- `x` is required
- `y` is optional
- everything after `y` goes into `operands`

Examples:

```elisp
(multiply-more 5)
;; => 5

(multiply-more 5 2)
;; => 10

(multiply-more 5 2 10)
;; => 100

(multiply-more 5 nil 10)
;; => 50
```

Use this sparingly. Optional and rest arguments are powerful, but they can make function calls harder to read if the argument order is not obvious.

## Documentation Strings

The first string in a function body is its documentation string.

```elisp
(defun do-some-math (x y)
  "Multiply the result of math expressions on X and Y."
  (* (+ x 20)
     (- y 10)))
```

Now Emacs can show documentation for the function:

```text
C-h f do-some-math
```

Documentation strings are not comments. They are part of the function definition and are visible through Emacs help.

By convention, argument names are written in uppercase inside docstrings:

```elisp
"Multiply the result of math expressions on X and Y."
```

This is why built-in Emacs documentation sometimes looks like it is shouting variable names. It is not angry; it is traditional.

## Multiline Documentation Strings

Docstrings can span multiple lines:

```elisp
(defun do-some-math (x y)
  "Multiply the result of math expressions on X and Y.

This function is intentionally silly. It exists to demonstrate
how function definitions and documentation strings work."
  (* (+ x 20)
     (- y 10)))
```

Do not indent wrapped docstring lines unless you want that indentation to appear in the help buffer.

This:

```elisp
"First line.
 Second line."
```

will show the leading space before `Second line`.

Use `M-q` inside the string to fill/wrap the paragraph cleanly.

## Anonymous Functions with `lambda`

Sometimes you need a function without giving it a permanent name.

Use `lambda`:

```elisp
(lambda (x y)
  (+ 100 x y))
```

This evaluates to a function value.

You can call it directly:

```elisp
((lambda (x y)
   (+ 100 x y))
 10 20)
;; => 130
```

This is not something you usually write for normal calls, but it demonstrates that functions can be values.

Anonymous functions are useful when passing behavior to another function, adding small hook functions, or defining callbacks.

## Why "Lambda"?

The name comes from lambda calculus, a mathematical system that influenced Lisp.

You do not need to understand lambda calculus to use Emacs Lisp. In practice, just remember:

```elisp
(lambda (...) ...)
```

means "create a function value here."

## Calling Function Values with `funcall`

Normal function calls look like this:

```elisp
(+ 2 2)
;; => 4
```

When the function itself is stored in a variable or passed as an argument, use `funcall`.

```elisp
(funcall '+ 2 2)
;; => 4
```

The quote before `+` matters:

```elisp
'+
```

It means "use the symbol `+` literally; do not try to evaluate it as a variable."

Function names and variable names live in separate namespaces in Emacs Lisp. A function named `named-version` does not automatically create a variable named `named-version`.

## Passing Functions to Functions

Define a function that accepts another function:

```elisp
(defun gimme-function (fun x)
  (message "The function was %S and the result is %S"
           fun
           (funcall fun x)))
```

Pass a lambda directly:

```elisp
(gimme-function (lambda (x) (1+ x)) 5)
;; message: result is 6
```

Store a function in a variable:

```elisp
(setq function-in-variable
      (lambda (x) (1+ x)))

(gimme-function function-in-variable 5)
;; message: result is 6
```

Define a named function:

```elisp
(defun named-version (x)
  (1+ x))
```

Pass the function by symbol:

```elisp
(gimme-function 'named-version 5)
;; message: result is 6
```

If you omit the quote:

```elisp
(gimme-function named-version 5)
```

Emacs tries to look up a variable named `named-version`, and you get an error.

## `apply`

`funcall` passes arguments directly:

```elisp
(funcall '+ 2 2)
;; => 4
```

`apply` passes a list of arguments:

```elisp
(apply '+ '(2 2))
;; => 4
```

This matters when you already have arguments in a list.

```elisp
(setq numbers '(5 2 10 7))

(apply 'multiply-many numbers)
;; => 700
```

Without `apply`, there is no simple direct way to spread that list into a normal function call.

Use `funcall` when you have individual arguments.

Use `apply` when you have a list of arguments.

## Commands and Interactive Functions

In Emacs, a command is an interactive function.

Commands can:

- appear in `M-x`
- be bound to keys
- prompt users for input
- accept prefix arguments

Normal functions do not automatically show up in `M-x`.

To make a function interactive, add an `interactive` form near the top of its body.

```elisp
(defun my-first-command ()
  "Show a friendly message."
  (interactive)
  (message "Hey, it worked!"))
```

Now you can run:

```text
M-x my-first-command
```

`C-h f my-first-command` will also describe it as an interactive function.

## Where `interactive` Goes

If a function has a docstring, `interactive` comes after the docstring:

```elisp
(defun my-command ()
  "Describe what this command does."
  (interactive)
  ...)
```

If there is no docstring, `interactive` should be the first expression in the body:

```elisp
(defun my-command ()
  (interactive)
  ...)
```

For user-facing functions, write a docstring. Your future self is a user too.

## Commands with Arguments

If a function requires arguments, simply adding `(interactive)` is not enough.

For example:

```elisp
(defun do-some-math (x y)
  (interactive)
  (* (+ x 20)
     (- y 10)))
```

This is interactive, but Emacs still does not know where `x` and `y` should come from.

Use an interactive code string to tell Emacs how to get the arguments.

## Prompt for Numbers

Use `N` to prompt for a number or accept a numeric prefix argument.

```elisp
(defun do-some-math (x y)
  "Prompt for X and Y, then show the computed result."
  (interactive "NX: \nNY: ")
  (message "The result is %d"
           (* (+ x 20)
              (- y 10))))
```

The string:

```elisp
"NX: \nNY: "
```

means:

- prompt for a number using `X: `
- newline separates the next interactive argument
- prompt for another number using `Y: `

Now you can run:

```text
M-x do-some-math
```

or bind it to a key.

## Bind a Command to a Key

Use `global-set-key` for a simple global binding:

```elisp
(global-set-key (kbd "C-c z") #'do-some-math)
```

Now:

```text
C-c z
```

will run `do-some-math`, prompt for `x` and `y`, and show the result.

The `#'` syntax is function quoting. For many practical cases here, it behaves like quoting a function symbol, and it communicates that you intend to refer to a function.

## Prompt for Strings

Use `M` to prompt for a string.

```elisp
(defun ask-favorite-fruit (fruit)
  "Ask for FRUIT and respond with a deeply unfair opinion."
  (interactive "MFavorite fruit: ")
  (message "%s? Interesting choice." fruit))
```

Run:

```text
M-x ask-favorite-fruit
```

Emacs prompts in the minibuffer.

## Prompt for Directories

Use `D` to prompt for an existing directory.

```elisp
(defun backup-directory (directory)
  "Pretend to back up DIRECTORY."
  (interactive "DSelect a path to back up: ")
  (message "Pretending to back up %s" directory))
```

Emacs uses your normal completion system for the prompt.

## Prompt for Commands

Use `C` to prompt for a command name.

```elisp
(defun run-a-command (command)
  "Ask for COMMAND and report what was selected."
  (interactive "CCommand: ")
  (message "You selected command: %S" command))
```

This gives you completion over command names, similar to `M-x`.

## Common Interactive Codes

Some useful interactive codes:

```text
N   number, or numeric prefix argument
p   numeric prefix argument only
M   string
f   existing file
F   possibly nonexistent file
d   directory name of current buffer
D   existing directory
b   existing buffer
B   buffer name, possibly nonexistent
c   character
C   command name
v   variable name
a   function name
i   ignore this argument
```

There are more. The Emacs Lisp manual has the full list under interactive codes.

## Prefix Arguments

Interactive commands can receive prefix arguments.

Users provide them with keys like:

```text
C-u
M-5
C-5
```

This lets one command change behavior based on how it was invoked.

For example, `N` can read a numeric prefix argument or prompt for a number if no prefix was provided.

Prefix arguments are one of Emacs' small superpowers: they let commands have extra behavior without requiring a separate command for every variant.

## Function Aliases

If you want to create an alias for a function, use `defalias`.

```elisp
(defalias 'do-maths #'do-some-math
  "British alias for `do-some-math'.")
```

Now `do-maths` refers to the same function.

This is better than defining a second wrapper function when you only need another name.

## Start a Project: Dotfiles Helpers

The series now starts building a real package: a small Emacs Lisp tool for managing dotfiles.

The project idea:

- manage an Org-based dotfiles repository
- tangle multiple Org files into real config files
- eventually manage symbolic links too
- do the work from Emacs without relying on tools like GNU Stow

This episode starts with two simple commands for tangling Org files.

## Configure the Dotfiles Folder

For now, use simple variables with `setq`.

```elisp
(setq dotfiles-folder "~/projects/code/emacs-from-scratch")

(setq dotfiles-org-files
      '("emacs.org"
        "desktop.org"))
```

`dotfiles-folder` is the root folder.

`dotfiles-org-files` is the list of Org files to tangle.

Later, these can become proper customizable variables.

## Tangle All Org Files

Use `dolist` to loop over the configured files:

```elisp
(defun dotfiles-tangle-org-files ()
  "Tangle all Org files in `dotfiles-org-files'."
  (interactive)
  (dolist (org-file dotfiles-org-files)
    (org-babel-tangle-file
     (expand-file-name org-file dotfiles-folder))))
```

`expand-file-name` builds the full path:

```elisp
(expand-file-name "emacs.org" dotfiles-folder)
```

If `dotfiles-folder` is:

```text
~/projects/code/emacs-from-scratch
```

then the result is:

```text
~/projects/code/emacs-from-scratch/emacs.org
```

Now run:

```text
M-x dotfiles-tangle-org-files
```

Emacs tangles each configured Org file.

## Tangle One Org File

It is useful to split the behavior into a smaller helper.

```elisp
(defun dotfiles-tangle-org-file (&optional org-file)
  "Tangle ORG-FILE.

When called interactively, prompt for ORG-FILE."
  (interactive "FOrg file: ")
  (org-babel-tangle-file
   (expand-file-name org-file dotfiles-folder)))
```

Now the all-files function can reuse it:

```elisp
(defun dotfiles-tangle-org-files ()
  "Tangle all Org files in `dotfiles-org-files'."
  (interactive)
  (dolist (org-file dotfiles-org-files)
    (dotfiles-tangle-org-file org-file)))
```

This is a small but important design habit: put one piece of behavior in one helper, then compose helpers together.

## Suppress Noisy Tangle Output

Tangling can write a lot of messages and may ask for confirmation before evaluating source blocks.

For a smoother command, temporarily bind a few variables while tangling:

```elisp
(defun dotfiles-tangle-org-file (&optional org-file)
  "Tangle ORG-FILE.

When called interactively, prompt for ORG-FILE."
  (interactive "FOrg file: ")
  (let ((inhibit-message t)
        (message-log-max nil)
        (org-confirm-babel-evaluate nil))
    (org-babel-tangle-file
     (expand-file-name org-file dotfiles-folder))))
```

`let` creates temporary bindings. This episode does not go deep on `let`; the next topic is variables and scope.

For now, the important idea is:

- suppress echo-area noise
- avoid filling `*Messages*`
- skip Org Babel confirmation prompts
- tangle the file

## A Starting Version

Here is the small starting point from this episode:

```elisp
(setq dotfiles-folder "~/projects/code/emacs-from-scratch")

(setq dotfiles-org-files
      '("emacs.org"
        "desktop.org"))

(defun dotfiles-tangle-org-file (&optional org-file)
  "Tangle ORG-FILE.

When called interactively, prompt for ORG-FILE."
  (interactive "FOrg file: ")
  (let ((inhibit-message t)
        (message-log-max nil)
        (org-confirm-babel-evaluate nil))
    (org-babel-tangle-file
     (expand-file-name org-file dotfiles-folder))))

(defun dotfiles-tangle-org-files ()
  "Tangle all Org files in `dotfiles-org-files'."
  (interactive)
  (dolist (org-file dotfiles-org-files)
    (dotfiles-tangle-org-file org-file))
  (message "Dotfiles are up to date"))
```

This is not a full package yet. It is the seed of one.

Future episodes can improve it by adding:

- real variable definitions
- customizable options
- scope and local bindings
- package structure
- link management
- error handling
- modes and commands

## What to Remember

The key ideas:

- `defun` defines named functions.
- Function bodies return the last expression's value.
- `&optional` arguments default to `nil`.
- `&rest` gathers remaining arguments into a list.
- Docstrings make functions discoverable through help.
- `lambda` creates anonymous function values.
- `funcall` calls a function value with direct arguments.
- `apply` calls a function with arguments from a list.
- `(interactive)` turns a function into a command.
- Interactive code strings tell Emacs how to prompt for arguments.
- Commands are the user-facing layer of Emacs Lisp.

Once you can define commands, you can start turning little bits of repeated workflow into real Emacs features.
