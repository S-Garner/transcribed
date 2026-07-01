# Learn Emacs Lisp 1: Emacs Lisp Basics

Emacs Lisp is the language used to configure and extend Emacs.

If you have edited an Emacs configuration, you have probably already written some Emacs Lisp, even if it was just:

```elisp
(setq inhibit-startup-message t)
```

or:

```elisp
(global-display-line-numbers-mode 1)
```

This first lesson gives you the basic mental model: how Emacs Lisp code is written, how expressions are evaluated, how function calls work, and how to experiment safely inside Emacs.

The later lessons build on these ideas, so the goal here is not to memorize every detail. The goal is to get comfortable reading and evaluating small expressions.

## Lisp Is Made of Expressions

Emacs Lisp code is made of expressions.

An expression is a piece of code that Emacs can evaluate to produce a value.

Some expressions evaluate to themselves:

```elisp
1
;; => 1

"hello"
;; => "hello"
```

These are called self-evaluating expressions.

Other expressions are lists:

```elisp
(+ 1 2)
```

When Emacs evaluates a list, it usually treats the first item as a function and the remaining items as arguments.

So:

```elisp
(+ 1 2)
```

means:

```text
call the function + with arguments 1 and 2
```

The result is:

```elisp
3
```

## Prefix Notation

Emacs Lisp uses prefix notation.

The operator or function comes first:

```elisp
(+ 1 2)
(* 3 4)
(- 10 5)
```

This may look strange if you are used to:

```text
1 + 2
```

but it has one nice property: every function call has the same shape.

```elisp
(function-name argument-1 argument-2 argument-3)
```

Nested expressions follow the same rule:

```elisp
(* (+ 1 2)
   (- 10 5))
;; => 15
```

Emacs evaluates the inner expressions first, then passes their results to the outer function.

## Parentheses Matter

In Emacs Lisp, parentheses are not decoration. They define structure.

This is a function call:

```elisp
(+ 1 2)
```

This is just the number `1`:

```elisp
1
```

This is a string:

```elisp
"hello"
```

If you add or remove parentheses, you change the meaning of the code.

Emacs helps you manage this with indentation and parenthesis matching. When in doubt, reindent the expression and look at its shape.

## Evaluate Code in Emacs

There are several ways to evaluate Emacs Lisp.

Evaluate the expression before point:

```text
C-x C-e
```

This is usually bound to `eval-last-sexp`.

For example, place point after this expression:

```elisp
(+ 2 2)
```

Then press:

```text
C-x C-e
```

Emacs prints the result in the echo area.

You can also evaluate a selected region:

```text
M-x eval-region
```

Or evaluate a whole buffer:

```text
M-x eval-buffer
```

## Use IELM

Emacs includes an Emacs Lisp REPL called IELM.

REPL means "read-eval-print loop." It reads an expression, evaluates it, prints the result, and waits for the next expression.

Start it with:

```text
M-x ielm
```

IELM gives you a prompt where you can type Emacs Lisp expressions and immediately see their results:

```elisp
ELISP> (+ 1 2)
3
```

This is often easier than evaluating expressions in a source buffer and looking for the result in the echo area. When learning small language features, a REPL is a friendly little laboratory.

Use it for small tests:

```elisp
(message "hello")
(+ 10 20)
(length "system crafters")
```

You can also use IELM to inspect values while reading examples:

```elisp
(type-of "hello")
(list 1 2 3)
(car '(alpha beta gamma))
```

The feedback loop is quick, and quick feedback makes Lisp much less mysterious.

## Symbols

A symbol is a name.

Examples:

```elisp
foo
message
setq
my-variable
```

Symbols can name variables, functions, commands, modes, hooks, and many other things.

When Emacs evaluates a symbol by itself, it treats it as a variable reference:

```elisp
fill-column
```

That means "look up the value of the variable named `fill-column`."

If the variable has no value, you get an error.

## Strings

Strings are text values:

```elisp
"hello"
"Emacs Lisp"
```

Strings evaluate to themselves:

```elisp
"hello"
;; => "hello"
```

You will use strings for messages, file paths, prompts, names, and configuration values.

Example:

```elisp
(message "Hello from Emacs Lisp")
```

`message` prints text in the echo area and writes it to the `*Messages*` buffer.

## Numbers

Numbers also evaluate to themselves:

```elisp
42
;; => 42

3.14
;; => 3.14
```

Basic arithmetic uses function-call syntax:

```elisp
(+ 1 2)
;; => 3

(* 5 10)
;; => 50
```

Numbers are covered in more detail in episode 2.

## Variables

Use `setq` to set a variable:

```elisp
(setq my-name "David")
```

Now evaluating `my-name` returns the string:

```elisp
my-name
;; => "David"
```

You can use that variable in another expression:

```elisp
(message "Hello, %s" my-name)
```

`setq` is common in Emacs configuration:

```elisp
(setq inhibit-startup-message t)
(setq visible-bell t)
(setq fill-column 80)
```

This means "set this variable to this value."

## `t` and `nil`

Emacs Lisp uses:

```elisp
t
nil
```

`t` means true.

`nil` means false.

You will see these constantly in configuration:

```elisp
(setq visible-bell t)
(setq make-backup-files nil)
```

In the first line, a feature is enabled.

In the second line, a feature is disabled.

Episode 2 goes deeper into truthiness, but the practical starting point is simple:

- use `t` for true/on/enabled
- use `nil` for false/off/disabled

## Lists

Lists are written with parentheses:

```elisp
(1 2 3)
("one" "two" "three")
```

But there is a catch.

When Emacs evaluates a list, it normally treats the first item as a function.

So this:

```elisp
(1 2 3)
```

tries to call `1` as a function, which is an error.

If you want a list as data, quote it.

## Quoting

Quote prevents evaluation.

```elisp
'(1 2 3)
;; => (1 2 3)
```

The leading quote means:

```text
do not evaluate the following expression; use it literally
```

This is essential for lists used as data.

For example:

```elisp
(setq my-list '(1 2 3))
```

Without the quote:

```elisp
(setq my-list (1 2 3))
```

Emacs would try to call `1` as a function.

Quoting also works with symbols:

```elisp
'hello
;; => hello
```

That means "the symbol `hello` itself," not "the value stored in the variable `hello`."

## Function Calls

A normal function call looks like this:

```elisp
(function-name arg1 arg2)
```

Examples:

```elisp
(message "Hello")
(length "hello")
(+ 1 2)
```

The first item is the function.

The rest are arguments.

This one:

```elisp
(length "hello")
```

calls `length` with one argument, the string `"hello"`.

The result is:

```elisp
5
```

## Functions Can Have Side Effects

Some functions primarily return values:

```elisp
(+ 1 2)
;; => 3
```

Some functions primarily do something:

```elisp
(message "Hello")
```

`message` returns a string, but the reason you call it is usually the side effect: showing text to the user.

Configuration code often uses side effects:

```elisp
(global-display-line-numbers-mode 1)
```

That expression enables a mode. The returned value is less important than the change it makes in Emacs.

## Define a Simple Function

You can define your own function with `defun`.

```elisp
(defun say-hello ()
  (message "Hello from my function"))
```

Evaluate that definition, then call it:

```elisp
(say-hello)
```

This is only a first taste. Episode 3 covers function definitions in detail, including arguments, documentation strings, anonymous functions, and interactive commands.

## Special Forms

Most lists are function calls, but some are special forms.

Special forms control evaluation in unusual ways.

For example:

```elisp
(setq my-name "David")
```

`setq` does not evaluate `my-name` as a variable. It treats `my-name` as the place to store a value.

Another common special form is `if`:

```elisp
(if t
    "yes"
  "no")
```

`if` does not evaluate both branches. It evaluates only the branch it needs.

You do not need to memorize all special forms right away. Just know that some core language constructs have their own evaluation rules.

## Comments

Comments start with semicolons:

```elisp
;; This is a comment.
(message "This code runs")
```

A common convention:

```elisp
;;; Big section or file header
;; Normal comment
; Small inline comment
```

For personal configuration, do not overthink it. Use comments when they explain why something is there.

## Formatting and Indentation

Emacs Lisp is much easier to read when indented correctly.

Good:

```elisp
(defun say-hello ()
  (message "Hello"))
```

Harder to read:

```elisp
(defun say-hello ()
(message "Hello"))
```

Emacs knows how to indent Lisp.

Indent the current line:

```text
TAB
```

Indent a region:

```text
M-x indent-region
```

If the indentation looks strange, your parentheses may be unbalanced. Lisp indentation is a surprisingly good smoke alarm.

## Getting Help

Emacs is self-documenting.

Describe a function:

```text
C-h f
```

Describe a variable:

```text
C-h v
```

Describe a key:

```text
C-h k
```

For example:

```text
C-h f message
```

shows documentation for the `message` function.

```text
C-h v fill-column
```

shows documentation for the `fill-column` variable.

This is one of the best ways to learn Emacs Lisp: inspect the functions and variables you are already using.

## Read Existing Configuration

A lot of Emacs Lisp learning happens by reading configuration snippets.

For example:

```elisp
(setq inhibit-startup-message t)
(global-display-line-numbers-mode 1)
(load-theme 'doom-one t)
```

Read these as:

- set the variable `inhibit-startup-message` to true
- call `global-display-line-numbers-mode` with argument `1`
- call `load-theme` with the symbol `doom-one` and true

That last one shows quoting again:

```elisp
'doom-one
```

The quote means "use the symbol `doom-one` as data."

Without the quote, Emacs would try to evaluate `doom-one` as a variable.

## A Tiny Configuration Example

Here is a small example that uses the basics:

```elisp
;; Hide the startup screen.
(setq inhibit-startup-message t)

;; Use spaces instead of tabs.
(setq-default indent-tabs-mode nil)

;; Set a simple helper function.
(defun open-init-file ()
  "Open my Emacs init file."
  (find-file user-init-file))

;; Bind it to a key.
(global-set-key (kbd "C-c i") #'open-init-file)
```

This includes:

- comments
- variable setting
- a function definition
- a documentation string
- a function call
- a quoted function reference
- a keybinding

You do not need to understand every detail yet. The important part is that the syntax is becoming recognizable.

## What to Remember

The core ideas:

- Emacs Lisp code is made of expressions.
- Many values, like numbers and strings, evaluate to themselves.
- Lists are usually function calls.
- The first item in a function call is the function.
- Emacs Lisp uses prefix notation.
- `setq` sets variables.
- `t` means true and `nil` means false.
- Quote prevents evaluation.
- Use `C-x C-e` and `M-x ielm` to experiment.
- Use `C-h f`, `C-h v`, and `C-h k` to inspect Emacs.

Once these basics feel less alien, the next step is learning the common data types and control-flow tools: numbers, strings, lists, conditionals, and loops.
