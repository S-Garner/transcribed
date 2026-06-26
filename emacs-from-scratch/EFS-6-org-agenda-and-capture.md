# Emacs From Scratch 6: Org Agenda, Capture, Refile, and Habits

The previous episode introduced Org Mode as a writing and outlining format. This episode goes deeper into the features that make Org useful as a planning system: agendas, task states, tags, capture templates, refiling, and habit tracking.

The goal is not to build a perfect productivity system in one pass. The goal is to understand the pieces Org gives you so you can shape a workflow that matches how you actually work.

By the end, you will know how to:

- collect tasks in Org files
- schedule tasks and set deadlines
- use the Org agenda as a daily view
- create custom TODO states
- filter tasks by state, tag, and effort
- define custom agenda views
- quickly capture tasks, journal entries, and metrics
- refile completed tasks into an archive
- track habits with Org's built-in habit module

## Visual Line Mode vs Auto Fill Mode

Before getting into planning, it is useful to clarify two line-wrapping features from the previous episode.

`visual-line-mode` visually wraps long lines without changing the file. The text looks wrapped on screen, but no new newline characters are inserted.

`auto-fill-mode` actually inserts line breaks as you type once a line reaches a certain width.

This matters for writing in Org. If you want your file contents to stay as continuous paragraphs, use `visual-line-mode`. If you want hard-wrapped lines in the file itself, use `auto-fill-mode`.

The visual setup from the previous episode used `visual-fill-column` to narrow the display and `visual-line-mode` to wrap the text visually inside that narrowed area.

## Create Org Files for Planning

Start by creating a folder for Org files:

```text
org-files/
```

Then create a task file:

```text
org-files/tasks.org
```

Inside it, add a few headings:

```org
* Active

** TODO Finish documentation

** TODO Merge the PR

* Backlog

** TODO Buy milk
```

The split between `Active` and `Backlog` is not required by Org. It is just a useful habit. Active tasks are things you expect to work on soon. Backlog tasks are things you want to remember but are not committed to doing immediately.

## Tell Org Agenda Which Files to Read

The Org agenda does not automatically scan every Org file on your computer. You need to tell it which files matter.

Add your Org files to `org-agenda-files`:

```elisp
(use-package org
  :custom
  (org-agenda-files
   '("~/projects/emacs-from-scratch/org-files/tasks.org")))
```

Adjust the path for your machine.

This matters because the agenda is an aggregate view. It pulls scheduled items, deadlines, TODOs, and timestamps from the files listed in `org-agenda-files`.

If you later add more files, include them in the list:

```elisp
(org-agenda-files
 '("~/projects/emacs-from-scratch/org-files/tasks.org"
   "~/projects/emacs-from-scratch/org-files/birthdays.org"
   "~/projects/emacs-from-scratch/org-files/habits.org"))
```

## Open the Org Agenda

Run:

```text
M-x org-agenda
```

Org shows a dispatcher menu. The most common entry is:

```text
a
```

for the agenda view.

You can jump directly to that view with:

```text
M-x org-agenda-list
```

At first, the agenda may be empty even if your task file has TODO items. That is because the agenda view shows scheduled items, deadlines, timestamps, and certain configured task views. Plain TODO headings do not automatically appear on today's calendar unless they are scheduled or matched by a custom agenda view.

## Schedule a Task

Scheduling means "show me this task starting on this date." It does not necessarily mean "this is due on this date."

Place point on a task and run:

```text
M-x org-schedule
```

This is commonly bound to:

```text
C-c C-s
```

Choose a date from the calendar prompt. Org inserts a line like:

```org
SCHEDULED: <2026-06-27 Sat>
```

Now the task appears in the agenda on that date.

This matters because scheduling turns a task from "something in a file" into "something I intend to consider on a date."

## Set a Deadline

A deadline means "this task should be completed by this date."

Place point on a task and run:

```text
M-x org-deadline
```

This is commonly bound to:

```text
C-c C-d
```

Org inserts a line like:

```org
DEADLINE: <2026-06-29 Mon>
```

Deadlines appear in the agenda before they are due. Org warns you ahead of time so the task does not surprise you on the final day.

Control the warning window with:

```elisp
(setq org-deadline-warning-days 7)
```

The default is often 14 days. Use whatever warning window fits your planning style.

## Show Completed Task Logs

The agenda can show a log of completed tasks. This is useful when you want your agenda to show what happened during the day, not only what is upcoming.

Add:

```elisp
(use-package org
  :custom
  (org-agenda-start-with-log-mode t)
  (org-log-done 'time)
  (org-log-into-drawer t))
```

Here is what each setting does:

- `org-agenda-start-with-log-mode` shows logged activity in the agenda.
- `org-log-done 'time` records a timestamp when a task is completed.
- `org-log-into-drawer` keeps log entries inside a collapsible drawer instead of cluttering the task body.

When you mark a task done, Org adds a closed timestamp:

```org
CLOSED: [2026-06-26 Fri 09:30]
```

Then the agenda can show that completed item in the day's log.

## View All TODO Items

Open the agenda dispatcher:

```text
M-x org-agenda
```

Then press:

```text
t
```

This shows TODO entries from your agenda files.

This is useful when you want a task list rather than a date-based calendar view.

## Track Dates That Are Not Tasks

Not everything belongs in TODO form. Birthdays, anniversaries, and other reminders are just dates you want to see.

Create:

```text
org-files/birthdays.org
```

Add:

```org
* Family

** Joe's birthday
<1990-10-10 Wed +1y>
```

The timestamp includes a repeater:

```text
+1y
```

That means the date repeats every year.

Add the file to `org-agenda-files`, then open the agenda around that date. Joe's birthday appears even though it is not a TODO task.

Repeaters can use different units:

- `+1d` for every day
- `+1w` for every week
- `+1m` for every month
- `+1y` for every year

This is useful for recurring dates that you want in your agenda without treating them like tasks.

## Desktop Notifications

Org itself can manage agenda data, but desktop notifications usually require an extra package.

One option is:

```elisp
(use-package org-wild-notifier)
```

This package can use Org agenda data to show operating-system notifications for upcoming items.

This is optional. Some people prefer notifications. Others prefer checking the agenda intentionally.

## Define Custom TODO States

Org's default states are usually `TODO` and `DONE`, but real workflows often need more detail.

For example, you may want a `NEXT` state for tasks that are ready to do next.

Add:

```elisp
(use-package org
  :custom
  (org-todo-keywords
   '((sequence "TODO(t)" "NEXT(n)" "|" "DONE(d!)")
     (sequence "BACKLOG(b)" "PLAN(p)" "READY(r)" "ACTIVE(a)" "REVIEW(v)" "WAIT(w@/!)" "HOLD(h)" "|" "COMPLETED(c)" "CANC(k@)"))))
```

The pipe character separates active states from completed states:

```text
TODO NEXT | DONE
```

The letters in parentheses are quick selection keys. For example, `NEXT(n)` means you can press `n` in the TODO state selector to choose `NEXT`.

The `!` and `@` markers control logging:

- `!` records a timestamp
- `@` prompts for a note

This matters because your task states should reflect how you think about work. A task that is merely captured is different from a task that is ready to do next.

## Change a Task State

Place point on a heading and run:

```text
C-c C-t
```

Org shows the available TODO states.

You can also cycle states with:

```text
S-RIGHT
S-LEFT
```

Once `NEXT` is configured, headings like this are recognized:

```org
** NEXT Merge the PR
```

## Define Custom Agenda Views

The default agenda views are useful, but custom views are where Org becomes tailored to your workflow.

Add:

```elisp
(use-package org
  :custom
  (org-agenda-custom-commands
   '(("n" "Next Tasks"
      todo "NEXT"
      ((org-agenda-overriding-header "Next Tasks")))

     ("d" "Dashboard"
      ((agenda "" ((org-deadline-warning-days 7)))
       (todo "NEXT"
             ((org-agenda-overriding-header "Next Tasks")))
       (tags-todo "agenda/ACTIVE"
                  ((org-agenda-overriding-header "Active Projects")))))

     ("w" "Work Tasks" tags-todo "+work-email")

     ("e" "Low Effort Tasks"
      tags-todo "+TODO=\"NEXT\"+Effort<15&+Effort>0"
      ((org-agenda-overriding-header "Low Effort Tasks")
       (org-agenda-max-todos 20))))))
```

This defines several custom agenda commands:

- `n` shows only `NEXT` tasks.
- `d` shows a dashboard with agenda, next tasks, and active projects.
- `w` shows work tasks but excludes email tasks.
- `e` shows low-effort `NEXT` tasks.

The syntax can look dense because Org agenda commands are nested lists. If you get errors, run:

```text
M-x check-parens
```

That helps find unbalanced parentheses in Lisp configuration.

## Filter With Tags

Tags let you classify tasks across files and headings.

Use Counsel's Org tag command:

```text
M-x counsel-org-tag
```

Or use Org's built-in tag command:

```text
C-c C-q
```

For example:

```org
** NEXT Merge the PR        :work:

** NEXT Reply to John's email     :work:email:
```

Tags become useful in agenda queries. This query includes `work` tasks but excludes `email` tasks:

```elisp
(tags-todo "+work-email")
```

The `+work` part means "must have the work tag." The `-email` part means "must not have the email tag."

## Define Common Tags

You can define a global list of tags that Org knows about:

```elisp
(use-package org
  :custom
  (org-tag-alist
   '((:startgroup)
     ("work" . ?w)
     ("personal" . ?p)
     (:endgroup)
     ("email" . ?e)
     ("planning" . ?n)
     ("publish" . ?P)
     ("note" . ?N))))
```

The character after `?` is the quick key used in Org's tag selection UI.

After changing this, restart `org-mode` in an existing Org buffer if the tag list does not update immediately:

```text
M-x org-mode
```

## Use Effort Estimates

Org headings can store properties. One useful property is `Effort`.

Set a property with:

```text
M-x org-set-property
```

Set effort directly with:

```text
M-x org-set-effort
```

Org stores the value in a property drawer:

```org
** NEXT Reply to John's email
:PROPERTIES:
:Effort:   5
:END:
```

Then custom agenda views can query by effort. For example:

```elisp
(tags-todo "+TODO=\"NEXT\"+Effort<15&+Effort>0")
```

This finds tasks that:

- are in the `NEXT` state
- have an effort value
- have effort less than 15

This is useful when you want to batch small tasks together.

## Refile Completed Tasks

As your task files grow, completed tasks can clutter the file. Refiling lets you move a heading to another file or heading.

Create an archive file:

```text
org-files/archive.org
```

Add headings:

```org
* 2026

** June
```

Configure refile targets:

```elisp
(use-package org
  :custom
  (org-refile-targets
   '(("~/projects/emacs-from-scratch/org-files/archive.org" :maxlevel . 3)
     ("~/projects/emacs-from-scratch/org-files/tasks.org" :maxlevel . 3))))
```

Now run:

```text
M-x org-refile
```

Org offers headings from the configured target files.

This matters because you can keep active task files focused while still preserving completed work elsewhere.

## Save Org Buffers After Refiling

Refiling can modify multiple Org files. To avoid forgetting to save one, add advice after `org-refile`:

```elisp
(advice-add 'org-refile :after 'org-save-all-org-buffers)
```

Advice lets you run extra behavior before, after, or around an existing function. In this case, every time `org-refile` finishes, Emacs saves all Org buffers.

This is a small workflow guardrail. It prevents refiles from sitting unsaved in multiple buffers.

## Capture Tasks Quickly

Capture templates are one of Org's most useful features. They let you quickly record a task, note, journal entry, or measurement without manually opening the target file.

Set a default notes file:

```elisp
(setq org-default-notes-file "~/projects/emacs-from-scratch/org-files/tasks.org")
```

Then define templates:

```elisp
(use-package org
  :custom
  (org-capture-templates
   `(("t" "Tasks")
     ("tt" "Task" entry
      (file+olp "~/projects/emacs-from-scratch/org-files/tasks.org" "Inbox")
      "* TODO %?\n  %U\n  %a\n"
      :empty-lines 1)

     ("j" "Journal")
     ("jj" "Journal" entry
      (file+olp+datetree "~/projects/emacs-from-scratch/org-files/journal.org")
      "\n* %<%I:%M %p> - Journal :journal:\n\n%?\n\n"
      :clock-in :clock-resume
      :empty-lines 1)

     ("m" "Metrics")
     ("mw" "Weight" table-line
      (file+headline "~/projects/emacs-from-scratch/org-files/metrics.org" "Weight")
      "| %U | %^{Weight} | %^{Notes} |"
      :kill-buffer t))))
```

Open capture with:

```text
M-x org-capture
```

The dispatcher shows the template keys. For example:

- `t t` captures a task
- `j j` captures a journal entry
- `m w` captures a weight measurement

## Understand Capture Template Markers

Capture templates use percent markers.

Common markers include:

- `%?` places the cursor there when the capture opens.
- `%U` inserts an inactive timestamp with date and time.
- `%a` inserts a link to where you were when capture started.
- `%<...>` inserts a formatted timestamp.
- `%^{Prompt}` asks for input in the minibuffer.

For example:

```org
* TODO %?
  %U
  %a
```

This creates a task, puts your cursor after the heading text, records the current timestamp, and links back to your current location.

That backlink is extremely useful when you are reading code or notes and need to capture "come back here later."

## Create an Inbox Heading

The task capture template above targets an `Inbox` heading:

```elisp
(file+olp "~/projects/emacs-from-scratch/org-files/tasks.org" "Inbox")
```

Make sure your task file contains:

```org
* Inbox
```

The inbox is a landing zone. Capture things quickly, then review and refile them later.

## Capture Journal Entries With a Date Tree

The journal template uses:

```elisp
file+olp+datetree
```

That creates a hierarchy by date:

```org
* 2026

** 2026-06 June

*** 2026-06-26 Friday

**** 09:15 AM - Journal :journal:
```

This is useful because you do not need to manually create daily headings. Org creates the structure as you capture entries.

## Capture Metrics Into a Table

The metrics template uses `table-line`:

```elisp
("mw" "Weight" table-line
 (file+headline "~/projects/emacs-from-scratch/org-files/metrics.org" "Weight")
 "| %U | %^{Weight} | %^{Notes} |"
 :kill-buffer t)
```

Create the target file:

```text
org-files/metrics.org
```

Add a heading and table header:

```org
* Weight

| Date | Weight | Notes |
|------+--------+-------|
```

When you run the capture template, Org asks for weight and notes, then appends a row to the table.

The `:kill-buffer t` option closes the target buffer after capture if it was opened only for the capture. That keeps your buffer list cleaner.

## Bind Capture to a Key

Capture is most useful when it is easy to reach.

Bind `org-capture` globally:

```elisp
(global-set-key (kbd "C-c c") 'org-capture)
```

You can also bind directly to a specific template:

```elisp
(global-set-key
 (kbd "C-c j")
 (lambda ()
   (interactive)
   (org-capture nil "jj")))
```

The first approach is usually enough because the capture dispatcher already gives you template prefixes.

## Resume the Last Capture

Org supports jumping back to the last stored capture with a universal argument:

```text
C-u M-x org-capture
```

If you bind `org-capture`, use the same idea with the keybinding:

```text
C-u C-c c
```

This is useful when you just captured something and immediately realize you want to revisit or continue it.

## Track Habits

Org includes a built-in habit tracker module.

Enable it:

```elisp
(require 'org-habit)

(add-to-list 'org-modules 'org-habit)

(setq org-habit-graph-column 60)
```

Then create:

```text
org-files/habits.org
```

Add the file to `org-agenda-files`.

Create a habit:

```org
* TODO Brush my teeth
SCHEDULED: <2026-06-26 Fri +1d>
:PROPERTIES:
:STYLE: habit
:END:
```

The important pieces are:

- a TODO heading
- a scheduled timestamp with a repeater
- `STYLE` set to `habit`

Now open the agenda:

```text
M-x org-agenda-list
```

Org shows a habit graph on the right side of the agenda. As you complete or miss the habit over time, the graph gives you a visual history.

## Full Example Configuration

Here is a combined configuration based on this episode. Adjust paths for your system.

```elisp
(use-package org
  :custom
  (org-agenda-files
   '("~/projects/emacs-from-scratch/org-files/tasks.org"
     "~/projects/emacs-from-scratch/org-files/birthdays.org"
     "~/projects/emacs-from-scratch/org-files/habits.org"))
  (org-agenda-start-with-log-mode t)
  (org-log-done 'time)
  (org-log-into-drawer t)
  (org-deadline-warning-days 7)
  (org-todo-keywords
   '((sequence "TODO(t)" "NEXT(n)" "|" "DONE(d!)")
     (sequence "BACKLOG(b)" "PLAN(p)" "READY(r)" "ACTIVE(a)" "REVIEW(v)" "WAIT(w@/!)" "HOLD(h)" "|" "COMPLETED(c)" "CANC(k@)")))
  (org-tag-alist
   '((:startgroup)
     ("work" . ?w)
     ("personal" . ?p)
     (:endgroup)
     ("email" . ?e)
     ("planning" . ?n)
     ("publish" . ?P)
     ("note" . ?N)))
  (org-refile-targets
   '(("~/projects/emacs-from-scratch/org-files/archive.org" :maxlevel . 3)
     ("~/projects/emacs-from-scratch/org-files/tasks.org" :maxlevel . 3)))
  (org-capture-templates
   `(("t" "Tasks")
     ("tt" "Task" entry
      (file+olp "~/projects/emacs-from-scratch/org-files/tasks.org" "Inbox")
      "* TODO %?\n  %U\n  %a\n"
      :empty-lines 1)
     ("j" "Journal")
     ("jj" "Journal" entry
      (file+olp+datetree "~/projects/emacs-from-scratch/org-files/journal.org")
      "\n* %<%I:%M %p> - Journal :journal:\n\n%?\n\n"
      :empty-lines 1)
     ("m" "Metrics")
     ("mw" "Weight" table-line
      (file+headline "~/projects/emacs-from-scratch/org-files/metrics.org" "Weight")
      "| %U | %^{Weight} | %^{Notes} |"
      :kill-buffer t)))
  (org-agenda-custom-commands
   '(("n" "Next Tasks"
      todo "NEXT"
      ((org-agenda-overriding-header "Next Tasks")))
     ("d" "Dashboard"
      ((agenda "" ((org-deadline-warning-days 7)))
       (todo "NEXT"
             ((org-agenda-overriding-header "Next Tasks")))
       (tags-todo "agenda/ACTIVE"
                  ((org-agenda-overriding-header "Active Projects")))))
     ("w" "Work Tasks" tags-todo "+work-email")
     ("e" "Low Effort Tasks"
      tags-todo "+TODO=\"NEXT\"+Effort<15&+Effort>0"
      ((org-agenda-overriding-header "Low Effort Tasks")
       (org-agenda-max-todos 20))))))

(require 'org-habit)
(add-to-list 'org-modules 'org-habit)
(setq org-habit-graph-column 60)

(advice-add 'org-refile :after 'org-save-all-org-buffers)

(global-set-key (kbd "C-c c") 'org-capture)
```

Example `tasks.org`:

```org
* Inbox

* Active

** NEXT Merge the PR :work:

** NEXT Reply to John's email :work:email:
:PROPERTIES:
:Effort:   5
:END:

** TODO Buy milk
:PROPERTIES:
:Effort:   20
:END:

* Backlog
```

Example `birthdays.org`:

```org
* Family

** Joe's birthday
<1990-10-10 Wed +1y>
```

Example `habits.org`:

```org
* TODO Brush my teeth
SCHEDULED: <2026-06-26 Fri +1d>
:PROPERTIES:
:STYLE: habit
:END:
```

Example `metrics.org`:

```org
* Weight

| Date | Weight | Notes |
|------+--------+-------|
```

## What This Gives You

Org Agenda gives you a control panel for your Org files. Scheduling and deadlines make work visible on dates. Custom TODO states let you model real workflows. Tags and effort estimates let you slice tasks by context and size. Capture templates give you a fast inbox for tasks, notes, journals, and measurements. Refile keeps old work out of the way. Habits give you a visual way to track repeated actions.

The main habit to build is simple: capture quickly, review regularly, and keep the agenda files small enough that you trust what they show.
