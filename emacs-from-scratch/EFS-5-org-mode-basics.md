# Emacs From Scratch 5: Org Mode Basics

The previous parts built a practical Emacs foundation: UI cleanup, better help, keybindings, project navigation, and Git workflows. This part introduces one of Emacs' most important packages: Org Mode.

Org Mode starts as an outliner and markup format, but it grows into much more than that. You can use it for notes, task lists, project planning, literate programming, exporting documents, and even generating configuration files.

This tutorial focuses on the basics:

- creating headings and outlines
- folding and unfolding sections
- formatting text
- inserting links
- creating tables
- writing plain lists and checklists
- adding source code blocks
- using basic TODO states
- making Org buffers look cleaner and more document-like

## Add Org to Your Package Archives

Emacs includes Org Mode, but the built-in version may lag behind the latest release. If you want the current Org package, add the Org package archive to your package setup.

Update `package-archives`:

```elisp
(setq package-archives
      '(("melpa" . "https://melpa.org/packages/")
        ("org" . "https://orgmode.org/elpa/")
        ("elpa" . "https://elpa.gnu.org/packages/")))
```

Then add Org with `use-package`:

```elisp
(use-package org)
```

This matters because Org is actively developed. Using the Org archive helps ensure you are not stuck on the older version bundled with your Emacs release.

You can check your Org version with:

```text
M-x org-version
```

## Create an Org File

Create a file ending in `.org`:

```text
C-x C-f hello.org
```

Emacs should automatically open it in `org-mode`.

The `.org` extension matters because Emacs uses it to choose the correct major mode. Once `org-mode` is active, Org-specific syntax highlighting and keybindings become available.

## Create Headings

Org documents are built around headings. A heading starts with one or more stars at the beginning of a line.

```org
* First heading

Here is some content.

* Second heading

More content here.

** Subheading

Nested content here.
```

The number of stars controls the heading level:

- `*` is level 1
- `**` is level 2
- `***` is level 3

This is the outline structure. Everything under a heading belongs to that heading until the next heading at the same or higher level.

## Fold and Unfold the Outline

Org Mode is an outliner, so you can collapse and expand sections.

Press:

```text
S-TAB
```

This cycles the visibility of the entire document.

Repeated presses usually move through views like:

- show only top-level headings
- show all headings
- show everything

This matters because Org files can become long. Folding lets you move from a detailed view to a high-level map of the document.

## Insert Headings Quickly

You do not need to type stars manually every time.

To insert a new heading at the same level after the current subtree, use:

```text
C-RET
```

To insert a heading immediately below the current heading, use:

```text
M-RET
```

These commands preserve the outline level for you.

If you are writing a long document or task list, this is much faster than manually counting stars.

## Move Headings Around

Org lets you reorganize documents structurally.

Move a heading up or down among siblings:

```text
M-UP
M-DOWN
```

Promote or demote a heading:

```text
M-LEFT
M-RIGHT
```

Depending on your terminal or operating system, these may appear as Alt plus arrow keys.

This matters because outlines are meant to be rearranged. Org makes structure editing cheap, so you can write first and organize later.

## Format Text

Org supports lightweight markup.

```org
*bold*
/italic/
_underline_
=verbatim=
~code~
```

When exported to HTML, PDF, or another format, this markup becomes formatted output.

This is one reason Org works well for writing. You can keep the source document readable while still producing nicely formatted output later.

## Insert Links

To insert a link, select some text and run:

```text
C-c C-l
```

Org asks for the URL, then uses the selected text as the description.

For example:

```org
[[https://orgmode.org][Org Mode homepage]]
```

To open the link at point, use:

```text
C-c C-o
```

This matters because Org documents can become research notebooks. Links let you connect notes to web pages, files, headings, or other resources.

## Create Tables

Org has excellent plain-text table support. Start a table with pipes:

```org
| Name      | Age | Uses Emacs |
|-----------+-----+------------|
| David     |  40 | yes        |
| Alexander |  31 | no         |
```

When your cursor is inside a table, press:

```text
TAB
```

Org realigns the table automatically.

This is useful because hand-aligning plain-text tables is tedious. Org keeps the table readable while still storing it as ordinary text.

## Create Plain Lists

Use hyphens for unordered lists:

```org
- First item
- Second item
- Third item
```

Use numbers for ordered lists:

```org
1. First item
2. Second item
3. Third item
```

While inside a list, press:

```text
M-RET
```

to create the next list item.

Plain lists are useful for notes that do not need to become full outline headings.

## Create Checklists

Add `[ ]` to a list item to make it a checkbox:

```org
- [ ] First task
- [ ] Second task
- [X] Completed task
```

Toggle a checkbox with:

```text
C-c C-x C-b
```

To continue a checkbox list, use:

```text
S-M-RET
```

Checklists are useful for small subtasks. Use them when you want a local checklist inside a note or TODO item without turning every item into a full Org heading.

## Add Source Blocks

Org can contain source code blocks.

```org
#+begin_src emacs-lisp
(defun my-elisp-func ()
  (message "Hello"))
#+end_src
```

The language name after `#+begin_src` tells Org how to highlight the block. For Emacs Lisp, use:

```org
#+begin_src emacs-lisp
```

Source blocks are a major Org feature. They can be used for notes, examples, executable code, and literate configuration. This tutorial only introduces the syntax; later workflows can use source blocks to generate real files or run code inside the document.

## Add TODO States

Any heading can become a task.

```org
* TODO Write project notes
```

Org recognizes `TODO` as a task state.

Toggle the task state with:

```text
C-c C-t
```

You can also cycle states with:

```text
S-RIGHT
S-LEFT
```

By default, Org commonly uses states like:

```org
TODO
DONE
```

Org's task system can be customized heavily, but this basic version is enough to start using Org as a to-do list.

## Make Org Buffers Look Better

Plain Org syntax is useful, but it can look noisy. Headings have stars, folded sections show `...`, formatting markers stay visible, and everything uses the same monospace font.

The next sections improve the visual presentation without changing the actual file format.

## Change the Fold Ellipsis

When Org folds a heading, it normally shows `...`. You can replace that with a cleaner symbol.

Add this to your Org config:

```elisp
(use-package org
  :custom
  (org-ellipsis " ▾"))
```

The file still contains ordinary Org text. This only changes how folded headings are displayed.

## Consider Hiding Emphasis Markers

Org can hide markup characters such as the `*` around bold text.

```elisp
(setq org-hide-emphasis-markers t)
```

With this enabled, `*bold*` appears as bold text without showing the surrounding stars.

The benefit is less visual noise. The tradeoff is editing can become slightly confusing because the hidden marker characters still exist. If you backspace near formatted text, the hidden markers can reappear or move.

This is a preference. If you like a clean document-like view, enable it. If you prefer always seeing the exact markup, leave it off.

## Replace Heading Stars With Bullets

The `org-bullets` package replaces heading stars with nicer bullet characters.

Add this:

```elisp
(use-package org-bullets
  :after org
  :hook (org-mode . org-bullets-mode)
  :custom
  (org-bullets-bullet-list '("◉" "○" "●" "○" "●" "○" "●")))
```

The stars are still in the file. `org-bullets` only changes their display.

This matters because deep Org outlines can become visually cluttered. Replacing repeated stars with cleaner bullets makes headings easier to scan.

## Scale Org Heading Sizes

Org exposes faces for each heading level, such as `org-level-1`, `org-level-2`, and so on. You can style those faces to make the document hierarchy clearer.

Add this:

```elisp
(dolist (face '((org-level-1 . 1.2)
                (org-level-2 . 1.1)
                (org-level-3 . 1.05)
                (org-level-4 . 1.0)
                (org-level-5 . 1.0)
                (org-level-6 . 1.0)
                (org-level-7 . 1.0)
                (org-level-8 . 1.0)))
  (set-face-attribute (car face) nil
                      :font "Cantarell"
                      :weight 'regular
                      :height (cdr face)))
```

This gives higher-level headings a larger variable-width font.

The point is not just decoration. Larger headings make the outline structure easier to read at a glance.

## Configure Fixed and Variable Pitch Fonts

Org documents often contain both prose and code. Prose usually reads better in a variable-width font, while code, tables, and blocks need a fixed-width font.

Configure both font categories:

```elisp
(set-face-attribute 'default nil
                    :font "Fira Code Retina"
                    :height 140)

(set-face-attribute 'fixed-pitch nil
                    :font "Fira Code Retina"
                    :height 140)

(set-face-attribute 'variable-pitch nil
                    :font "Cantarell"
                    :height 160
                    :weight 'regular)
```

Use fonts installed on your own system. If `Fira Code Retina` or `Cantarell` is not available, replace them.

## Use Variable Pitch Mode in Org

To make Org prose use the variable-width font, enable `variable-pitch-mode` in Org buffers.

Define a setup function:

```elisp
(defun efs/org-mode-setup ()
  (org-indent-mode)
  (variable-pitch-mode 1)
  (visual-line-mode 1))
```

Attach it to Org Mode:

```elisp
(use-package org
  :hook (org-mode . efs/org-mode-setup))
```

This makes Org buffers feel more like document editors while keeping the underlying file plain text.

## Keep Code and Tables Fixed Width

Once `variable-pitch-mode` is enabled, some things that need alignment may look wrong. Source blocks and tables should stay fixed width.

Set those Org faces to inherit from `fixed-pitch`:

```elisp
(set-face-attribute 'org-block nil :inherit 'fixed-pitch)
(set-face-attribute 'org-code nil :inherit '(shadow fixed-pitch))
(set-face-attribute 'org-table nil :inherit 'fixed-pitch)
(set-face-attribute 'org-verbatim nil :inherit '(shadow fixed-pitch))
(set-face-attribute 'org-special-keyword nil :inherit '(font-lock-comment-face fixed-pitch))
(set-face-attribute 'org-meta-line nil :inherit '(font-lock-comment-face fixed-pitch))
(set-face-attribute 'org-checkbox nil :inherit 'fixed-pitch)
```

This gives you the best of both styles:

- prose uses a comfortable document font
- code and tables still align correctly

## Replace List Hyphens With Dots

If you prefer cleaner list bullets, you can visually replace hyphens with dots using font lock.

Add this inside your Org setup:

```elisp
(font-lock-add-keywords 'org-mode
                        '(("^ *\\([-]\\) "
                           (0 (prog1 ()
                                (compose-region (match-beginning 1)
                                                (match-end 1)
                                                "•"))))))
```

This does not change the file contents. It only changes how list hyphens are displayed in Org buffers.

This is a cosmetic choice. Keep normal hyphens if you prefer the source text to look exactly like the display.

## Center Org Text With Visual Fill Column

When writing prose, full-width editor windows can be hard to read. Long lines stretch across the screen, and the text starts at the far left edge.

`visual-fill-column` lets you set a visual text width and center it.

Add the package:

```elisp
(use-package visual-fill-column
  :hook (org-mode . efs/org-mode-visual-fill))
```

Define the setup function:

```elisp
(defun efs/org-mode-visual-fill ()
  (setq visual-fill-column-width 100
        visual-fill-column-center-text t)
  (visual-fill-column-mode 1))
```

This makes Org buffers feel more like a writing environment. The text wraps at a readable width and is centered in the window.

`visual-line-mode` handles visual wrapping, while `visual-fill-column-mode` controls the width and centering.

## Full Example Org Configuration

Here is a combined version of the Org configuration from this tutorial:

```elisp
(defun efs/org-mode-setup ()
  (org-indent-mode)
  (variable-pitch-mode 1)
  (visual-line-mode 1))

(defun efs/org-mode-visual-fill ()
  (setq visual-fill-column-width 100
        visual-fill-column-center-text t)
  (visual-fill-column-mode 1))

(use-package org
  :hook (org-mode . efs/org-mode-setup)
  :custom
  (org-ellipsis " ▾"))

(use-package org-bullets
  :after org
  :hook (org-mode . org-bullets-mode)
  :custom
  (org-bullets-bullet-list '("◉" "○" "●" "○" "●" "○" "●")))

(use-package visual-fill-column
  :hook (org-mode . efs/org-mode-visual-fill))

(dolist (face '((org-level-1 . 1.2)
                (org-level-2 . 1.1)
                (org-level-3 . 1.05)
                (org-level-4 . 1.0)
                (org-level-5 . 1.0)
                (org-level-6 . 1.0)
                (org-level-7 . 1.0)
                (org-level-8 . 1.0)))
  (set-face-attribute (car face) nil
                      :font "Cantarell"
                      :weight 'regular
                      :height (cdr face)))

(set-face-attribute 'org-block nil :inherit 'fixed-pitch)
(set-face-attribute 'org-code nil :inherit '(shadow fixed-pitch))
(set-face-attribute 'org-table nil :inherit 'fixed-pitch)
(set-face-attribute 'org-verbatim nil :inherit '(shadow fixed-pitch))
(set-face-attribute 'org-special-keyword nil :inherit '(font-lock-comment-face fixed-pitch))
(set-face-attribute 'org-meta-line nil :inherit '(font-lock-comment-face fixed-pitch))
(set-face-attribute 'org-checkbox nil :inherit 'fixed-pitch)

(font-lock-add-keywords 'org-mode
                        '(("^ *\\([-]\\) "
                           (0 (prog1 ()
                                (compose-region (match-beginning 1)
                                                (match-end 1)
                                                "•"))))))
```

If you want hidden emphasis markers, add this setting to the `:custom` section of the `use-package org` block:

```elisp
(org-hide-emphasis-markers t)
```

That would make the Org block look like this:

```elisp
(use-package org
  :hook (org-mode . efs/org-mode-setup)
  :custom
  (org-ellipsis " ▾")
  (org-hide-emphasis-markers t))
```

## What This Gives You

At this point, Org Mode can serve as a structured writing environment. You can write outlines, collapse and expand sections, insert links, create tables, maintain checklists, include source code, and mark headings as TODO or DONE.

The visual configuration makes Org feel less like raw markup and more like a focused document editor, while still keeping everything as plain text.

The next natural step is to use Org for task and project management: custom TODO workflows, agenda views, deadlines, scheduled tasks, capture templates, and project planning.
