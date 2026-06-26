# Emacs From Scratch 4: Projects and Git

The previous parts focused on making Emacs comfortable: a cleaner UI, better help, better completion, and better keybindings. This part starts turning that setup into a real daily workflow for software projects.

Most programming work happens inside projects. A project might be a Git repository, a package folder, a documentation site, or any directory with related files and commands. Emacs becomes much more useful when it understands that context.

This tutorial covers two major packages:

- Projectile, for project navigation and project commands
- Magit, for working with Git from inside Emacs

It also covers supporting tools:

- `counsel-projectile`, for Ivy/Counsel integration
- Ripgrep, for fast project search
- directory-local variables, for project-specific settings
- Forge, for GitHub issues and pull requests inside Emacs

## Why Project Tools Matter

You can open files manually with `C-x C-f`, and that works fine for small directories. But once a project grows, you need faster ways to answer questions like:

- What files are in this project?
- Where is this symbol or string used?
- How do I run this project?
- How do I run its tests?
- What changed since my last commit?
- Which branch am I on?
- What do I need to stage, commit, rebase, or push?

Projectile answers the project-navigation questions. Magit answers the Git questions. Together, they remove a lot of context switching.

## Add Projectile

Projectile is a project interaction library for Emacs. It detects projects using signals such as Git repositories, package files, and other common project markers.

Add this to your config:

```elisp
(use-package projectile
  :diminish projectile-mode
  :config
  (projectile-mode)
  :custom
  (projectile-project-search-path '("~/projects/"))
  (projectile-switch-project-action #'projectile-dired)
  :bind-keymap
  ("C-c p" . projectile-command-map))
```

Adjust the search path for your machine:

```elisp
(projectile-project-search-path '("~/code/" "~/work/"))
```

Here is what each part does:

- `(projectile-mode)` enables Projectile globally.
- `projectile-project-search-path` tells Projectile where to look for projects.
- `projectile-switch-project-action` controls what happens after switching projects.
- `projectile-dired` opens the project root in Dired.
- `:bind-keymap` binds Projectile's command map under `C-c p`.

The command map matters because Projectile has many commands. Instead of binding each one manually, `C-c p` becomes the prefix for Projectile actions.

## Explore the Projectile Command Map

After evaluating the Projectile config, press:

```text
C-c p
```

If you have Which Key enabled, you should see a panel of Projectile commands.

Common commands include:

- `C-c p f` to find a file in the current project
- `C-c p p` to switch projects
- `C-c p u` to run the project
- `C-c p P` to test the project
- `C-c p E` to edit directory-local variables

This is where the earlier Which Key setup starts paying off. You do not need to memorize Projectile all at once. You can press the prefix and discover commands as you need them.

## Find Files in a Project

Run:

```text
C-c p f
```

Projectile shows files from the current project. This is similar to `Ctrl-P` style file search in editors like VS Code or Sublime Text.

This matters because project file search is usually faster than navigating directory-by-directory. If you know part of a filename, you can jump straight there.

## Switch Between Projects

Run:

```text
C-c p p
```

Projectile lists known projects. Selecting one switches to it and runs the action configured by `projectile-switch-project-action`.

In the config above, switching projects opens Dired at the project root:

```elisp
(projectile-switch-project-action #'projectile-dired)
```

That is a good default because it gives you a project-level file listing immediately.

## Run and Test Projects

Projectile can run project-level commands.

Run a project:

```text
C-c p u
```

Test a project:

```text
C-c p P
```

The first time you run either command in a project, Projectile asks for the command. For example, in a Node.js project you might use:

```bash
npm start
```

or:

```bash
npm test
```

Projectile runs these commands in a compilation buffer. That is useful because you can keep editing while the command output stays visible in another window.

If you need to stop a running process, kill the compilation buffer. Emacs will ask whether to kill the running process attached to it.

## Add Ivy Integration With `counsel-projectile`

Projectile works without Ivy, but the completion UI is better when it uses the same Ivy/Counsel interface as the rest of your config.

First, tell Projectile to use Ivy:

```elisp
(use-package projectile
  :custom
  (projectile-completion-system 'ivy))
```

If you already have a Projectile block, merge that `:custom` setting into it.

Then add `counsel-projectile`:

```elisp
(use-package counsel-projectile
  :after (counsel projectile)
  :config
  (counsel-projectile-mode))
```

This gives Projectile commands better Ivy integration and richer actions.

For example, when you are in a Projectile completion list, press:

```text
M-o
```

Ivy shows available actions for the selected item. Depending on the command, these actions might open a file another way, delete a file, switch project context, or run another related operation.

## Search a Project With Ripgrep

Ripgrep is a fast command-line search tool. It is especially useful in large repositories.

You need the external `rg` command installed on your system. The Emacs package integration depends on that executable existing.

On many Linux systems:

```bash
sudo apt install ripgrep
```

On macOS with Homebrew:

```bash
brew install ripgrep
```

Then install the Emacs package:

```elisp
(use-package rg)
```

With `counsel-projectile` enabled, search the current project with:

```text
M-x counsel-projectile-rg
```

This gives you live project search through Ivy. Type a search term, and matching files appear in the minibuffer.

This is one of the most useful project workflows in Emacs. When you need to understand a codebase, fast search is usually the first tool you reach for.

## Keep Search Results in a Buffer

Ivy search results normally disappear after you select an item. Sometimes you want the result list to stay around so you can inspect several matches.

While an Ivy search is active, press:

```text
C-c C-o
```

This creates an Ivy Occur buffer with the current results.

From that buffer, you can move through results and press `RET` to jump to a match. If Evil Collection is enabled, `q` usually closes the result buffer.

This matters in large codebases because you often need to inspect several matches before deciding what to change.

## Use Directory-Local Variables

Sometimes a project needs its own Emacs settings. For example, one project might use a different test command, formatter, indentation style, or Projectile run command.

Emacs supports directory-local variables through a file named:

```text
.dir-locals.el
```

Projectile can help create or edit this file:

```text
C-c p E
```

For example, you can set a project-specific run command:

```elisp
((nil . ((projectile-project-run-cmd . "npm start"))))
```

This means that buffers under this directory should use `"npm start"` as the Projectile run command.

Directory-local variables are applied per buffer. They do not globally change Emacs. When you open a file inside that directory, Emacs applies the relevant local values for that buffer.

## Reload Directory-Local Variables

After editing `.dir-locals.el`, existing buffers may not pick up the change immediately.

You can reload them by reverting the buffer:

```text
M-x revert-buffer
```

You can also evaluate:

```elisp
(hack-dir-local-variables)
```

from `M-:`.

When Emacs loads directory-local variables, it may ask whether a value is safe. That safety prompt exists because a project could contain malicious local variables. For trusted projects, you can accept the value. For unknown projects, be cautious.

## Add Magit

Magit is a Git interface inside Emacs. It is one of the strongest reasons to use Emacs for software development.

Add this:

```elisp
(use-package magit
  :commands (magit-status magit-get-current-branch)
  :custom
  (magit-display-buffer-function #'magit-display-buffer-same-window-except-diff-v1))
```

The main command is:

```text
M-x magit-status
```

Magit also commonly binds this to:

```text
C-x g
```

The status buffer gives you a live Git dashboard for the current repository.

The custom display setting makes most Magit buffers reuse the current window except for diffs. That is a workflow preference. If you prefer Magit's defaults, leave that `:custom` setting out.

## Read the Magit Status Buffer

Open Magit status:

```text
C-x g
```

You will see information such as:

- current branch
- current HEAD commit
- upstream branch
- untracked files
- unstaged changes
- staged changes
- recent commits

Put your cursor on a section and press:

```text
TAB
```

Magit expands or collapses that section. For changed files, this lets you inspect diffs directly inside the status buffer.

## Stage and Unstage Changes

In the status buffer:

```text
s
```

stages the item at point.

```text
u
```

unstages the item at point.

Uppercase versions apply more broadly:

```text
S
U
```

Depending on where point is, these can stage or unstage all relevant changes.

You can stage at different levels:

- on a file heading, stage the file
- on a hunk, stage the hunk
- on a selected region inside a diff, stage only that region

This is the core Magit workflow. It turns staging into a review process: inspect each change, stage what belongs in the commit, leave out what does not.

## Commit Changes

After staging changes, press:

```text
c c
```

The first `c` opens the commit menu. The second `c` creates a regular commit.

Magit opens a commit-message buffer. Write your message, then confirm with:

```text
C-c C-c
```

To cancel the commit, use:

```text
C-c C-k
```

The commit buffer usually shows the staged diff in another window. That is useful when writing a commit message because you can check exactly what the commit contains.

## Use Magit's Command Menus

In Magit status, press:

```text
?
```

Magit shows available commands. You will see menus for:

- branch
- commit
- diff
- fetch
- pull
- push
- rebase
- stash
- cherry-pick

You do not have to press `?` every time. It is mainly a discovery tool. Once you know a command, you can run it directly from the status buffer.

## Create or Switch Branches

Open the branch menu:

```text
b
```

From there, common actions include:

- `b` to checkout an existing branch
- `c` to create a branch
- `s` to create a spin-off branch

The spin-off command is especially useful when you accidentally committed to the wrong branch. It creates a new branch with commits that are ahead of the upstream branch and moves your original branch back.

This saves a common Git cleanup sequence: create a branch, switch back, reset the original branch, and verify everything.

## Stash Changes

If Git refuses to switch branches because you have local changes, use a stash.

Open the stash menu:

```text
z
```

Create a stash:

```text
z
```

Pop a stash later:

```text
z p
```

Apply a stash without deleting it:

```text
z a
```

Magit also lets you inspect stash contents directly from the status buffer.

## Push Changes

Open the push menu:

```text
P
```

Common push commands include:

- `P p` to push to the remote
- `P u` to push to upstream
- `P e` to push elsewhere

If you rebased or rewrote history, you may need a force push:

```text
P -f p
```

Magit shows the force option before running the push, which makes the operation explicit.

Authentication depends on your Git setup. If you use SSH, Git may prompt for your SSH key passphrase. If that prompt appears somewhere unexpected, make sure your Emacs and SSH agent setup are working together correctly.

## Fix Up a Previous Commit

Magit can quickly add new changes to an earlier commit.

Stage the change you want to add, then open the commit menu:

```text
c
```

Use instant fixup:

```text
f
```

Magit asks which commit to fix up. Choose the target commit and confirm.

This is useful when you notice that a new change really belongs in a previous commit. Instead of manually running an interactive rebase and moving commits around, Magit does the cleanup for you.

## Rebase Interactively

Open the rebase menu:

```text
r
```

Start an interactive rebase:

```text
i
```

Magit asks which commit to rebase from. Choose the commit where the rebase should begin.

In the rebase buffer, you can mark commits with actions like:

- pick
- squash
- fixup
- reword
- drop

For example, press:

```text
s
```

on a commit to squash it.

Then confirm with:

```text
C-c C-c
```

If a conflict happens during a rebase, return to the rebase menu. Magit will show continue and abort options for the in-progress operation.

## Add Forge for GitHub Issues and Pull Requests

Forge extends Magit with support for forge-based Git hosting services such as GitHub and GitLab. It lets you work with issues, pull requests, labels, and repository metadata from inside Emacs.

Add this:

```elisp
(use-package forge
  :after magit)
```

The first install may take a little while because Forge may compile an Emacs SQLite helper.

Forge needs API authentication before it can talk to GitHub or another hosting service. Follow Forge's documentation for token creation and storage. For GitHub, this typically involves creating a personal access token and storing it where the `ghub` library can read it.

## Pull Forge Data

Inside a repository, run:

```text
M-x forge-pull
```

Forge fetches issues, pull requests, and related metadata.

After that, open Magit status:

```text
C-x g
```

You should see additional sections for issues and pull requests when data is available.

## Create a Pull Request With Forge

Run:

```text
M-x forge-create-pullreq
```

Forge asks for:

- source branch
- target branch
- pull request title and description

After submitting, the pull request appears on the remote service.

This is useful because it keeps the Git workflow in one place. You can create the branch, push it, and open the pull request without leaving Emacs.

## Create an Issue With Forge

Run:

```text
M-x forge-create-issue
```

Write the issue title and body, then submit with:

```text
C-c C-c
```

Forge creates the issue on the remote service.

This is not required for a good Git workflow, but it is valuable if you want Emacs to be your main project-management interface too.

## Full Example Configuration Additions

Here are the main additions from this part:

```elisp
(use-package projectile
  :diminish projectile-mode
  :config
  (projectile-mode)
  :custom
  (projectile-project-search-path '("~/projects/"))
  (projectile-switch-project-action #'projectile-dired)
  (projectile-completion-system 'ivy)
  :bind-keymap
  ("C-c p" . projectile-command-map))

(use-package counsel-projectile
  :after (counsel projectile)
  :config
  (counsel-projectile-mode))

(use-package rg)

(use-package magit
  :commands (magit-status magit-get-current-branch)
  :custom
  (magit-display-buffer-function #'magit-display-buffer-same-window-except-diff-v1))

(use-package forge
  :after magit)
```

And an example `.dir-locals.el` for a Node.js project:

```elisp
((nil . ((projectile-project-run-cmd . "npm start")
         (projectile-project-test-cmd . "npm test"))))
```

## What This Gives You

Projectile gives Emacs project awareness: file search, project switching, project commands, and project-local configuration. Ripgrep gives you fast search across a repository. Magit gives you a complete Git interface that is faster and more inspectable than many command-line workflows. Forge brings issues and pull requests into that same environment.

Together, these packages move Emacs from "text editor with custom keybindings" toward "project workspace." You can open files, search code, run commands, stage hunks, commit, rebase, push, and create pull requests without leaving Emacs.
