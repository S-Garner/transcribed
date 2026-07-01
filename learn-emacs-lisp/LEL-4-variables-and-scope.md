# Learn Emacs Lisp 4: Variables and Scope

Variables look simple at first: a name points to a value.

In Emacs Lisp, there is more to the story. Variables can be global, buffer-local, dynamically scoped, temporarily rebound, declared for documentation, or exposed through Emacs' customization interface.

This tutorial covers:

- what variables are
- `set`, `setq`, and `setq-default`
- defining variables with `defvar`
- buffer-local variables
- scope
- `let` and `let*`
- dynamic scoping
- customization variables with `defcustom`
- applying these ideas to the dotfiles package project

## What Is a Variable?

A variable is an association between a name and a value.

In Emacs Lisp, the name is a symbol.

For example:

```elisp
tab-width
```

can be bound to:

```elisp
4
```

When you evaluate:

```elisp
tab-width
```

Emacs looks up the value currently bound to the symbol `tab-width`.

That value may depend on where the lookup happens. A variable can have one value globally, another value in a specific buffer, and a temporary value inside a `let` expression.

That is what makes scope worth understanding.

## Set a Variable with `set`

The most direct way to set a variable is `set`.

```elisp
(set 'tab-width 4)
```

`set` takes a symbol and a value.

The quote matters:

```elisp
'tab-width
```

means "use the symbol `tab-width` literally."

Without the quote, Emacs would try to evaluate `tab-width` first and use its value as the thing to set.

You can inspect `set` with:

```text
C-h o set
```

Some primitive functions are better inspected with `describe-symbol`, which is bound to `C-h o`.

## Set a Variable with `setq`

Most Emacs Lisp code uses `setq` instead of `set`.

```elisp
(setq tab-width 4)
```

`setq` means "set quoted." It saves you from writing the quote yourself.

These are equivalent in normal use:

```elisp
(set 'tab-width 4)
(setq tab-width 4)
```

`setq` can also set several variables at once:

```elisp
(setq tab-width 4
      fill-column 80
      visible-bell t)
```

The arguments are pairs:

```text
variable value variable value ...
```

This is common in configuration files.

## Variables Can Be Created by Setting Them

You do not have to define a variable before setting it.

```elisp
(setq i-do-not-exist 5)
```

After evaluating that, `i-do-not-exist` exists as a variable.

You can inspect it with:

```text
C-h v i-do-not-exist
```

This flexibility is useful, but it also means typos can silently create new variables. If you meant to set `tab-width` but typed `tba-width`, Emacs may happily create `tba-width` for you. A tiny chaos portal, very on brand.

## Define a Variable with `defvar`

Use `defvar` when you want to declare a variable and give it documentation.

```elisp
(defvar am-i-documented "yes"
  "A small variable used to demonstrate `defvar'.")
```

This gives the variable:

- a name
- an initial value
- a documentation string

Now:

```text
C-h v am-i-documented
```

shows the documentation.

Use `defvar` for variables that are part of a package or for configuration values you want to document clearly.

## `defvar` Does Not Always Override Existing Values

If a variable already has a value, `defvar` does not replace it.

```elisp
(setq am-i-documented "no")

(defvar am-i-documented "yes"
  "A small variable used to demonstrate `defvar'.")

am-i-documented
;; => "no"
```

This behavior is intentional.

It allows users to set a variable before a package loads. When the package later declares the variable with `defvar`, the user's value is preserved.

This matters for packages that read variables during initialization.

When developing, if you really want to re-evaluate the `defvar` and reset the value, use:

```text
M-x eval-defun
```

with point inside the `defvar` expression.

## When to Use `setq` vs `defvar`

Use `setq` when:

- configuring existing variables
- creating quick personal configuration values
- setting temporary or private values in your init file

Use `defvar` when:

- writing package-level variables
- documenting a variable for users
- declaring a variable that other code may reference

For personal config, `setq` is often enough. For reusable code, prefer explicit variable definitions.

## Buffer-Local Variables

Some variables can have different values in different buffers.

For example, `tab-width` is buffer-local.

One buffer might use:

```elisp
tab-width
;; => 2
```

while another uses:

```elisp
tab-width
;; => 4
```

This is useful because different file types often need different settings.

For example:

- C might use a tab width of 4.
- JavaScript might use a tab width of 2.
- Makefiles may need literal tabs.

Run:

```text
C-h v tab-width
```

The help buffer will tell you that it is a buffer-local variable.

## Set a Buffer-Local Value

Use `setq-local`:

```elisp
(setq-local tab-width 4)
```

This sets `tab-width` only in the current buffer.

If you switch to another buffer, `tab-width` may have a different value.

You can also make an arbitrary variable buffer-local in the current buffer:

```elisp
(setq some-value 2)
(setq-local some-value 4)
```

After `setq-local`, future `setq` calls for that variable in the same buffer update the buffer-local value:

```elisp
(setq some-value 5)
```

In that buffer, `some-value` is now `5`. In another buffer, the global value may still be `2`.

## Variables That Exist Only in One Buffer

You can create a variable only in the current buffer:

```elisp
(setq-local only-buffer-local "maybe")
```

In the current buffer:

```elisp
only-buffer-local
;; => "maybe"
```

In another buffer, the variable may not exist at all.

This can be useful for major modes, minor modes, and buffer-specific state.

## Make a Variable Buffer-Local Everywhere

Use `make-variable-buffer-local`:

```elisp
(defvar not-local-yet t
  "Example variable.")

(make-variable-buffer-local 'not-local-yet)
```

Now the variable is considered buffer-local in all buffers.

Buffers still share the global default until a specific buffer sets its own local value.

This is common in package code:

```elisp
(defvar my-mode-state nil
  "Internal state for `my-mode'.")

(make-variable-buffer-local 'my-mode-state)
```

## Set the Default Value

For buffer-local variables, use `setq-default` to set the global default:

```elisp
(setq-default tab-width 2)
```

This affects buffers that do not have their own buffer-local value.

It does not override an existing buffer-local value in the current buffer.

For example:

```elisp
(setq-local tab-width 4)
(setq-default tab-width 2)

tab-width
;; => 4 in the current buffer
```

Another buffer without a local value may see:

```elisp
tab-width
;; => 2
```

## Be Careful Using Buffer-Local Values Globally

This can be surprising:

```elisp
(setq-default evil-shift-width tab-width)
```

If `tab-width` is buffer-local, this uses the value from the current buffer.

So the default for `evil-shift-width` may depend on which buffer evaluated the expression.

If you need the default value of a buffer-local variable, use `default-value`:

```elisp
(default-value 'tab-width)
```

Then:

```elisp
(setq-default evil-shift-width (default-value 'tab-width))
```

This avoids accidentally copying a buffer-specific value into a global default.

## What Is Scope?

Scope is the region of code where a variable binding is active.

At the broadest level, Emacs has a global scope. A variable set globally can be accessed from almost anywhere.

Buffers can add a narrower scope for buffer-local variables.

`let` can add an even narrower temporary scope.

When Emacs looks up a variable, it checks the most specific active binding first. If it does not find one, it keeps looking outward.

In simplified terms:

```text
local let binding
buffer-local binding
global binding
```

The actual rules have details, but this model is enough to explain most behavior you will see while configuring Emacs.

## Why Global Variables Can Be Dangerous

Global variables are useful for configuration and state.

But they can make code less predictable if you mutate them casually.

Example:

```elisp
(setq x 0)

(defun do-the-loop ()
  (message "Starting the loop from %d" x)
  (while (< x 5)
    (message "x is %d" x)
    (setq x (1+ x)))
  (message "Done"))
```

The first call loops from `0` to `4`.

The second call starts from `5`, because the previous call changed the global `x`.

That is probably not what you wanted.

Use local variables for temporary state.

## Local Variables with `let`

`let` creates temporary variable bindings.

```elisp
(let ((y 5)
      (z 10))
  (* y z))
;; => 50
```

The bindings exist only inside the body of the `let`.

The general shape is:

```elisp
(let ((name value)
      (name value))
  body...)
```

The double parentheses are there because `let` takes a list of bindings, and each binding is itself a list.

Use `let` to avoid polluting global scope.

## Fix the Loop with `let`

Use a local `x`:

```elisp
(defun do-the-loop ()
  (let ((x 0))
    (message "Starting the loop from %d" x)
    (while (< x 5)
      (message "x is %d" x)
      (setq x (1+ x)))
    (message "Done")))
```

Now every call starts from `0`, because each call creates a fresh local binding.

The global `x`, if it exists, is not changed.

## `let` Can Temporarily Override Globals

If a global variable exists, `let` can temporarily rebind it:

```elisp
(setq x 5)

(let ((x 15))
  x)
;; => 15

x
;; => 5
```

Inside the `let`, `x` is `15`.

Outside the `let`, `x` is still `5`.

This is very useful when you want to change package behavior temporarily.

## `let` vs `let*`

Bindings in a plain `let` are established in parallel.

This does not work:

```elisp
(let ((y 5)
      (z (+ y 5)))
  (* y z))
```

The binding for `z` cannot see the `y` being defined in the same binding list.

Use `let*` when later bindings depend on earlier bindings:

```elisp
(let* ((y 5)
       (z (+ y 5)))
  (* y z))
;; => 50
```

You can think of `let*` as nested `let` forms:

```elisp
(let ((y 5))
  (let ((z (+ y 5)))
    (* y z)))
```

Practical rule: if a variable in your binding list needs a previous variable from the same list, use `let*`.

## Dynamic Scope

By default, Emacs Lisp uses dynamic scope unless lexical binding is enabled.

Dynamic scope means a variable's value can depend on where a function is called, not only where it was defined.

Example:

```elisp
(setq x 5)

(defun do-some-math (y)
  (+ x y))

(do-some-math 10)
;; => 15
```

`x` is a free variable inside `do-some-math`.

Now call the function inside a `let` that rebinds `x`:

```elisp
(let ((x 15))
  (do-some-math 10))
;; => 25
```

The function uses the dynamically active value of `x`.

Afterward:

```elisp
(do-some-math 10)
;; => 15
```

because the global `x` is still `5`.

## Why Dynamic Scope Matters

Dynamic scope can be surprising if you come from languages with lexical scope by default.

It can also be useful.

For example, you can temporarily change the behavior of a package function by rebinding a variable around the call:

```elisp
(let ((org-confirm-babel-evaluate nil))
  (org-babel-tangle-file "emacs.org"))
```

`org-babel-tangle-file` checks `org-confirm-babel-evaluate`. Inside this `let`, it sees `nil`, so it does not prompt.

The global value is not permanently changed.

This pattern appears often in Emacs Lisp:

```elisp
(let ((some-package-setting temporary-value))
  (call-some-package-function))
```

It is a clean way to adjust behavior for one operation.

Lexical scope is also available in Emacs Lisp, and it is often better for many kinds of program structure. That deserves its own focused discussion. For now, understand dynamic scope because you will see it constantly in Emacs code.

## Customization Variables

Emacs has a customization system.

Variables defined with `defcustom` appear in the customization UI and include metadata about how users should edit them.

Basic shape:

```elisp
(defcustom my-custom-variable 42
  "A customizable variable."
  :type 'integer
  :group 'my-package)
```

This is similar to `defvar`, but with extra metadata.

Important keywords include:

- `:type` describes the expected value for the customization UI
- `:group` places the variable in a customization group
- `:options` can list possible values
- `:set` can define custom behavior when setting the value
- `:get` can define custom behavior when reading the value
- `:initialize` controls initialization behavior
- `:local` can make the variable buffer-local

The `:type` metadata mostly helps the customization UI present the right editor. It is not the same as general runtime type checking.

## Define a Custom Group

For package code, define a customization group:

```elisp
(defgroup dotfiles nil
  "Manage dotfiles from Emacs."
  :group 'tools)
```

Then define variables in that group:

```elisp
(defcustom dotfiles-folder "~/.dotfiles"
  "The path to the dotfiles folder."
  :type 'string
  :group 'dotfiles)
```

Users can find these settings with:

```text
M-x customize-group RET dotfiles RET
```

## Set Custom Variables Correctly

You can set many custom variables with `setq`, and it often appears to work.

But if a custom variable defines a custom setter with `:set`, `setq` will not call that setter.

Use:

```elisp
(customize-set-variable 'tab-width 2)
```

This uses the customization machinery and invokes custom setter behavior when present.

If you use `use-package`, prefer `:custom`:

```elisp
(use-package emacs
  :custom
  (tab-width 2))
```

For package configuration:

```elisp
(use-package org
  :custom
  (org-directory "~/notes"))
```

`use-package`'s `:custom` section sets custom variables in the appropriate way.

## How to Tell Whether a Variable Is Customizable

Use:

```text
C-h v variable-name
```

If the variable is customizable, the help buffer will say so and usually provide a customization link.

You can also check programmatically:

```elisp
(custom-variable-p 'tab-width)
;; non-nil

(custom-variable-p 'org--file-cache)
;; nil
```

`custom-variable-p` may return metadata rather than plain `t`, so treat any non-nil result as true.

## Apply This to the Dotfiles Project

In episode 3, the dotfiles helper used plain variables:

```elisp
(setq dotfiles-folder "~/projects/code/emacs-from-scratch")

(setq dotfiles-org-files
      '("emacs.org"
        "desktop.org"))
```

Now turn those into customization variables.

First define a group:

```elisp
(defgroup dotfiles nil
  "Manage dotfiles from Emacs."
  :group 'tools)
```

Then define the folder:

```elisp
(defcustom dotfiles-folder "~/.dotfiles"
  "The path to the dotfiles folder."
  :type 'string
  :group 'dotfiles)
```

Define the list of Org files:

```elisp
(defcustom dotfiles-org-files '("emacs.org" "desktop.org")
  "List of Org files to tangle in `dotfiles-folder'."
  :type '(repeat string)
  :group 'dotfiles)
```

`(repeat string)` tells the customization UI this should be a list of strings.

Now users can configure the package without editing the source directly:

```text
M-x customize-group RET dotfiles RET
```

## Updated Dotfiles Helper

The helper from episode 3 can now use custom variables:

```elisp
(defgroup dotfiles nil
  "Manage dotfiles from Emacs."
  :group 'tools)

(defcustom dotfiles-folder "~/.dotfiles"
  "The path to the dotfiles folder."
  :type 'string
  :group 'dotfiles)

(defcustom dotfiles-org-files '("emacs.org" "desktop.org")
  "List of Org files to tangle in `dotfiles-folder'."
  :type '(repeat string)
  :group 'dotfiles)

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

This is still small, but it is becoming package-shaped:

- user options live in a customization group
- internal behavior uses local bindings
- commands expose user-facing behavior

## What to Remember

The key ideas:

- Variables are symbol-to-value bindings.
- `setq` is the normal way to set variables in configuration.
- `defvar` declares and documents variables.
- `defvar` does not override existing values.
- `setq-local` creates or sets a buffer-local binding.
- `make-variable-buffer-local` makes a variable buffer-local by default.
- `setq-default` changes the default value for buffer-local variables.
- Scope determines which binding is visible.
- `let` creates temporary local bindings.
- `let*` lets later bindings use earlier bindings.
- Emacs Lisp uses dynamic scope by default unless lexical binding is enabled.
- `defcustom` defines user-facing configuration variables.
- Use `customize-set-variable` or `use-package :custom` for custom variables when setter behavior matters.

Variables are not just boxes for values. In Emacs Lisp, they are one of the main ways packages, buffers, commands, and user configuration communicate with each other.
