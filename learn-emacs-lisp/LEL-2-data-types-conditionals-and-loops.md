# Learn Emacs Lisp 2: Data Types, Conditionals, and Loops

This episode continues the Emacs Lisp fundamentals by looking at the values you work with most often and the control-flow tools you use to make decisions.

The focus is on:

- truth and falsehood
- equality predicates
- numbers and characters
- strings, lists, and arrays
- association lists and property lists
- logic expressions
- conditionals
- loops

This is foundation work. It may feel like a lot of small pieces, but these are the pieces that make normal Emacs customization and package code readable.

## Follow Along with IELM

Emacs includes an interactive Emacs Lisp REPL called IELM.

Start it with:

```text
M-x ielm
```

IELM lets you type Emacs Lisp expressions and immediately see their results.

For example:

```elisp
(+ 1 2)
```

returns:

```elisp
3
```

This is often easier than evaluating expressions in a source buffer and looking for the result in the echo area. When learning small language features, a REPL is a friendly little laboratory.

## True and False

Many languages have a Boolean type with values like `true` and `false`.

Emacs Lisp uses:

```elisp
t
nil
```

`t` means true.

`nil` means false.

Both are symbols:

```elisp
(type-of t)
;; => symbol

(type-of nil)
;; => symbol
```

Predicates are functions that answer yes-or-no questions. In Emacs Lisp, they normally return `t` or `nil`.

For example:

```elisp
(integerp 1)
;; => t

(integerp "1")
;; => nil
```

Most predicate names end in `p`, as in `integerp`, `stringp`, `listp`, and `sequencep`.

## Truthiness

In Emacs Lisp, only `nil` is false.

Everything else is true, including:

```elisp
0
""
"hello"
42
'anything
[]
```

This surprises people coming from languages where `0` or empty strings are false.

In Emacs Lisp:

```elisp
(if 0
    "true"
  "false")
;; => "true"
```

The empty list is also written as `nil`:

```elisp
()
;; => nil
```

So false and the empty list are the same value.

## Equality

Emacs Lisp has several equality predicates. They answer slightly different questions.

### `eq`

`eq` asks whether two values are the same object.

```elisp
(eq 1 1)
;; => t

(eq 3.1 3.1)
;; => nil

(eq "thing" "thing")
;; => nil
```

This is identity comparison, not general value comparison.

Use `eq` when you want to know whether two variables refer to the exact same object or when comparing symbols:

```elisp
(eq 'foo 'foo)
;; => t
```

### `eql`

`eql` is like `eq`, but it handles numbers more usefully.

```elisp
(eql 1 1)
;; => t

(eql 3.1 3.1)
;; => t

(eql "thing" "thing")
;; => nil
```

### `equal`

`equal` compares values structurally.

```elisp
(equal 1 1)
;; => t

(equal 3.1 3.1)
;; => t

(equal "thing" "thing")
;; => t

(equal '(1 2 3) '(1 2 3))
;; => t
```

When in doubt, especially for strings and lists, `equal` is usually the one you want.

## Numbers

Emacs Lisp has integers and floating-point numbers.

Integers:

```elisp
1
-10
0
```

Floats:

```elisp
3.14
-1.2
1.0
```

Numbers are self-evaluating:

```elisp
1
;; => 1

3.14
;; => 3.14
```

## Arithmetic

Emacs Lisp uses prefix notation. The operator comes first:

```elisp
(+ 5 5)
;; => 10

(- 5 5)
;; => 0

(* 5 5)
;; => 25

(/ 5 5)
;; => 1
```

Nested arithmetic is just nested function calls:

```elisp
(* (+ 3 2)
   (- 10 5))
;; => 25
```

Prefix notation can look odd at first, but it makes nested expressions regular and easy to reindent.

## Remainders

For integer remainder, use `%`:

```elisp
(% 11 5)
;; => 1
```

For floating-point remainder, use `mod`:

```elisp
(mod 11.1 5)
;; => 1.0999999999999996
```

Remainders are useful when mapping arbitrary numbers into a fixed range. For example, `(% n 5)` always produces a number from `0` through `4`.

## Increment and Decrement

Use `1+` and `1-`:

```elisp
(1+ 5)
;; => 6

(1- 5)
;; => 4
```

These are handy in loops and counters.

## Rounding Floats

Several functions convert floats to integers:

```elisp
(truncate 1.2)
;; => 1

(truncate -1.2)
;; => -1

(floor 1.2)
;; => 1

(floor -1.2)
;; => -2

(ceiling 1.2)
;; => 2

(round 1.5)
;; => 2
```

The difference matters for negative numbers.

- `truncate` moves toward zero.
- `floor` moves downward.
- `ceiling` moves upward.
- `round` moves to the nearest integer.

## Number Predicates

Check for integer values:

```elisp
(integerp 1)
;; => t

(integerp 1.1)
;; => nil
```

Check for floats:

```elisp
(floatp 1.1)
;; => t

(floatp 1)
;; => nil
```

Check for any number:

```elisp
(numberp 1)
;; => t

(numberp 1.1)
;; => t

(numberp "1")
;; => nil
```

Check for zero:

```elisp
(zerop 0)
;; => t

(zerop 0.0)
;; => t

(zerop 1)
;; => nil
```

## Number Comparisons

Use the usual comparison operators:

```elisp
(= 5 5)
;; => t

(> 5 4)
;; => t

(< 4 5)
;; => t

(>= 5 5)
;; => t

(<= 5 5)
;; => t
```

`=` can compare more than two numbers:

```elisp
(= 1 1 1 1)
;; => t
```

Use `min` and `max` to find boundaries:

```elisp
(max 1 7 3 -1)
;; => 7

(min 1 7 3 -1)
;; => -1
```

## Characters

Characters are represented as integers, but Emacs Lisp gives them readable syntax.

```elisp
?A
;; => 65

?a
;; => 97
```

Special characters use escape syntax:

```elisp
?\n
;; newline character

?\t
;; tab character
```

Unicode characters can be written in several ways:

```elisp
?\N{LATIN SMALL LETTER A WITH GRAVE}
```

Control and meta character syntax appears in lower-level key handling:

```elisp
?\C-c
?\M-x
```

Most of the time, for keybindings, you will use `kbd` instead:

```elisp
(kbd "C-c")
```

## Character Comparisons

Use `char-equal`:

```elisp
(char-equal ?a ?a)
;; => t

(char-equal ?a 97)
;; => t
```

Case sensitivity is affected by `case-fold-search`.

When `case-fold-search` is non-nil, case is ignored in searches and some matching operations:

```elisp
(setq case-fold-search t)

(char-equal ?a ?A)
;; => t
```

With it disabled:

```elisp
(setq case-fold-search nil)

(char-equal ?a ?A)
;; => nil
```

Be aware of this variable when string or character matching behaves unexpectedly.

## Sequences

Strings, lists, and arrays are sequences.

Check with `sequencep`:

```elisp
(sequencep "hello")
;; => t

(sequencep '(1 2 3))
;; => t

(sequencep [1 2 3])
;; => t

(sequencep 1)
;; => nil
```

`nil` also counts as a sequence because it represents the empty list:

```elisp
(sequencep nil)
;; => t
```

## Sequence Length

Use `length`:

```elisp
(length "hello!")
;; => 6

(length '(1 2 3))
;; => 3

(length [1 2 3 4])
;; => 4

(length nil)
;; => 0
```

## Sequence Elements

Use `elt` to get an element by zero-based index:

```elisp
(elt "hello" 1)
;; => ?e

(elt '(3 2 1) 2)
;; => 1

(elt [1 2 3 4] 2)
;; => 3
```

Zero-based means the first element is index `0`, not index `1`.

Asking for out-of-range elements behaves differently depending on the sequence type. Lists may return `nil`; strings and arrays generally signal an error.

## Strings

Strings are sequences of characters:

```elisp
"hello"
```

Strings are self-evaluating:

```elisp
"hello"
;; => "hello"
```

You can write multiline strings directly:

```elisp
"hello
system crafters"
```

Escape special characters with backslash:

```elisp
"hello\nsystem crafters"
```

Use `\\` when you need a literal backslash.

## Create Strings

Create a repeated-character string:

```elisp
(make-string 5 ?!)
;; => "!!!!!"
```

Create a string from characters:

```elisp
(string ?h ?e ?l ?l ?o ?!)
;; => "hello!"
```

## String Predicates

```elisp
(stringp "test")
;; => t

(stringp 1)
;; => nil

(string-or-null-p nil)
;; => t

(char-or-string-p ?a)
;; => t

(char-or-string-p "a")
;; => t
```

Because strings are also arrays and sequences:

```elisp
(arrayp "hello")
;; => t

(sequencep "hello")
;; => t

(listp "hello")
;; => nil
```

## String Comparisons

Compare strings with:

```elisp
(string-equal "hello" "hello")
;; => t

(string-lessp "mellow" "yellow")
;; => t

(string-greaterp "yellow" "mellow")
;; => t
```

String ordering is based on sort order, not string length alone.

For example:

```elisp
(string-lessp "hell" "hello")
;; => t
```

because the shorter string sorts before the longer one when the shared prefix is the same.

## String Operations

Get part of a string:

```elisp
(substring "hello" 0 4)
;; => "hell"

(substring "hello" 1)
;; => "ello"
```

Join strings:

```elisp
(concat "hello" " " "system" " " "crafters")
;; => "hello system crafters"
```

Split strings:

```elisp
(split-string "hello system crafters")
;; => ("hello" "system" "crafters")
```

Split with a regular expression:

```elisp
(split-string "hello system crafters!" "[ !]")
;; => ("hello" "system" "crafters" "")
```

Remove empty results by passing non-nil for the omit-nulls argument:

```elisp
(split-string "hello system crafters!" "[ !]" t)
;; => ("hello" "system" "crafters")
```

Like character comparison, string matching can be affected by `case-fold-search`.

## Formatting Strings

Use `format` to build strings from values:

```elisp
(format "Hello %d %s" 100 "system crafters")
;; => "Hello 100 system crafters"
```

Common format specifiers:

- `%s` inserts a printed representation suitable for strings
- `%d` inserts an integer
- `%f` inserts a floating-point value

Use `message` to print formatted text to the echo area and `*Messages*` buffer:

```elisp
(message "This is %d" 5)
;; prints: This is 5
```

`message` is useful for diagnostics while developing.

## Lists

Lists are one of the central data types in Emacs Lisp.

A list is built from cons cells.

A cons cell is a pair of values:

```elisp
(cons 1 2)
;; => (1 . 2)
```

The left side is called the `car`.

The right side is called the `cdr`.

```elisp
(car '(1 . 2))
;; => 1

(cdr '(1 . 2))
;; => 2
```

The names are old Lisp terminology. They look strange for a while, then one day your brain stops objecting. Mostly.

## Mutating Cons Cells

Store a cons cell:

```elisp
(setq some-cons (cons 1 2))
```

Change the `car`:

```elisp
(setcar some-cons 3)

some-cons
;; => (3 . 2)
```

Change the `cdr`:

```elisp
(setcdr some-cons 4)

some-cons
;; => (3 . 4)
```

Mutation is powerful, but use it carefully. Many Emacs Lisp operations are easier to reason about when you build new values rather than modifying old ones in place.

## Lists from Cons Cells

A proper list is a chain of cons cells ending in `nil`.

This:

```elisp
(cons 1
      (cons 2
            (cons 3
                  (cons 4 nil))))
```

prints as:

```elisp
(1 2 3 4)
```

You can also cons a value onto an existing list:

```elisp
(cons 1 '(2 3 4))
;; => (1 2 3 4)
```

To combine two lists, use `append`:

```elisp
(append '(1 2 3) '(4 5 6))
;; => (1 2 3 4 5 6)
```

Do not use `cons` when your intent is to concatenate two lists:

```elisp
(cons '(1 2 3) '(4 5 6))
;; => ((1 2 3) 4 5 6)
```

That creates a list whose first element is another list.

## List Predicates

```elisp
(listp '(1 2 3))
;; => t

(listp "hello")
;; => nil

(consp '(1 2 3))
;; => t

(consp '(1 . 2))
;; => t
```

Remember that:

```elisp
(listp nil)
;; => t
```

because `nil` is the empty list.

## Association Lists

An association list, or alist, is a list of key-value pairs.

```elisp
(setq some-alist
      '((one . 1)
        (two . 2)
        (three . 3)))
```

Get values with `alist-get`:

```elisp
(alist-get 'one some-alist)
;; => 1

(alist-get 'four some-alist)
;; => nil
```

Get the whole pair by key with `assq`:

```elisp
(assq 'one some-alist)
;; => (one . 1)
```

Find a pair by value with `rassq`:

```elisp
(rassq 1 some-alist)
;; => (one . 1)
```

Set an alist value with `setf`:

```elisp
(setf (alist-get 'one some-alist) 5)

(alist-get 'one some-alist)
;; => 5
```

Alists are very common in Emacs configuration.

## Property Lists

A property list, or plist, is a flat list of alternating keys and values:

```elisp
'(:one 1 :two 2)
```

Get a value with `plist-get`:

```elisp
(plist-get '(:one 1 :two 2) :one)
;; => 1
```

Add or change a value with `plist-put`:

```elisp
(plist-put '(:one 1 :two 2) :three 3)
;; => (:one 1 :two 2 :three 3)
```

You will see plists often in APIs where keyword-style options are convenient.

## Arrays and Vectors

Arrays store values next to each other in memory, making indexed access fast.

Vectors are the most common array-like syntax you will write directly:

```elisp
[1 2 3 4]
```

Get an element:

```elisp
(elt [1 2 3 4] 2)
;; => 3
```

Set an element with `aset`:

```elisp
(setq some-array [1 2 3 4])
(aset some-array 1 5)

some-array
;; => [1 5 3 4]
```

Strings can also be treated like arrays of characters:

```elisp
(setq some-string "hello")
(aset some-string 0 ?m)

some-string
;; => "mello"
```

Fill an array:

```elisp
(setq some-array [1 2 3])
(fillarray some-array 6)

some-array
;; => [6 6 6]
```

## Logic Expressions

Logic expressions combine truthy and falsey values.

Use `not` to invert truth:

```elisp
(not t)
;; => nil

(not 3)
;; => nil

(not nil)
;; => t
```

Use `and` when all expressions must be true:

```elisp
(and 1 2 3 "foo")
;; => "foo"
```

`and` returns the last truthy value if everything succeeds. It stops early and returns `nil` as soon as it sees a false value:

```elisp
(and nil "never evaluated")
;; => nil
```

Use `or` when any expression may be true:

```elisp
(or nil 'something t)
;; => something
```

`or` returns the first truthy value and stops evaluating the rest.

This short-circuit behavior is useful. You can put expensive or side-effecting expressions later, and they will only run if needed.

## `if`

`if` chooses between two branches:

```elisp
(if t
    5
  10)
;; => 5
```

With a false condition:

```elisp
(if nil
    5
  (message "false branch")
  (+ 2 2))
;; => 4
```

`if` is an expression, so it returns the value of the branch it evaluates.

The true branch can contain only one expression. The false branch can contain multiple expressions.

If you need multiple expressions in the true branch, use `progn`.

## `progn`

`progn` evaluates several expressions and returns the last value:

```elisp
(progn
  (message "hello")
  (+ 2 2))
;; => 4
```

Use it inside `if` when the true branch needs multiple steps:

```elisp
(if t
    (progn
      (message "hey, it is true")
      5)
  10)
;; => 5
```

## Inline Conditionals

Because `if` returns a value, you can use it inside assignments:

```elisp
(setq tab-width
      (if (string-equal (format-time-string "%A") "Monday")
          3
        2))
```

That example sets `tab-width` differently on Mondays. Please use this power responsibly; your future self and coworkers have suffered enough.

## `when` and `unless`

Use `when` when you only care about the true case:

```elisp
(when (> 2 1)
  'foo)
;; => foo
```

Use `unless` when you only care about the false case:

```elisp
(unless (> 1 2)
  'foo)
;; => foo
```

Both forms can contain multiple expressions and return the last expression's value:

```elisp
(when (> 2 1)
  (message "yes")
  (+ 2 2))
;; => 4
```

If their condition does not match, they return `nil`.

## `cond`

`cond` handles multiple branches.

```elisp
(setq a 1)

(cond
 ((= a 1)
  "equal to one")
 ((> a 1)
  "greater than one")
 (t
  "something else"))
;; => "equal to one"
```

Each branch is a list. The first element is the test. The remaining elements are evaluated when that test succeeds.

The first matching branch wins.

The final `t` branch is the catch-all case.

Like `if`, `cond` returns the value of the last expression in the branch it evaluates.

## Pattern Matching with `pcase`

Emacs Lisp also has `pcase`, a powerful pattern-matching form.

This episode only mentions it, because `pcase` deserves separate treatment. It is useful when you want to match values by shape, not just test simple conditions.

For now, know that if `cond` starts to feel awkward for matching structured values, `pcase` may be the tool you want.

## Loops

Emacs Lisp gives you several ways to repeat work:

- `while`
- `dotimes`
- `dolist`
- recursion

## `while`

Use `while` when you want to loop as long as a condition remains true.

```elisp
(setq my-loop-counter 0)

(while (< my-loop-counter 5)
  (message "I'm looping: %d" my-loop-counter)
  (setq my-loop-counter (1+ my-loop-counter)))
```

This prints values from `0` through `4`.

You are responsible for changing the loop state. If you forget to update `my-loop-counter`, you get an infinite loop. Emacs will not gently tap you on the shoulder.

## `dotimes`

Use `dotimes` when you know how many times to loop.

```elisp
(dotimes (count 5)
  (message "I'm looping more easily: %d" count))
```

`count` receives values from `0` through `4`.

This is cleaner than manually creating and incrementing a counter.

## `dolist`

Use `dolist` to loop over a list:

```elisp
(dolist (item '(1 2 3 4 5))
  (message "Item: %d" item))
```

The loop body runs once for each item.

This is one of the most common loop forms you will use in Emacs Lisp configuration, because many configuration variables are lists.

## Recursion

Recursion means a function calls itself.

```elisp
(defun efs/recursion-test (counter limit)
  (when (< counter limit)
    (message "I'm looping via recursion: %d" counter)
    (efs/recursion-test (1+ counter) limit)))

(efs/recursion-test 0 5)
```

This works, but recursion is not usually the best default loop style in Emacs Lisp.

Some Lisp languages rely heavily on recursion and optimize tail-recursive calls. Emacs Lisp does not generally give you the same safety for deep recursion, so recursive loops over large lists can hit limits or perform poorly.

For most practical Emacs Lisp, start with `dolist`, `dotimes`, or `while`.

## What to Remember

The key ideas from this episode:

- `t` is true; `nil` is false and also the empty list.
- Everything except `nil` is truthy.
- Use `equal` for general value comparison.
- Numbers use prefix arithmetic: `(+ 1 2)`.
- Characters are integers with readable syntax like `?a`.
- Strings, lists, and vectors are sequences.
- Lists are chains of cons cells.
- Alists and plists are common key-value structures.
- `and` and `or` short-circuit.
- `if`, `when`, `unless`, and `cond` all return values.
- Use `dotimes` for counted loops and `dolist` for list loops.

The next natural step is functions: defining them, passing arguments, returning values, and using them to build real Emacs customizations.
