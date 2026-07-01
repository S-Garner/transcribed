# Managing Dotfiles 3: Manage Symlinks with GNU Stow

Manual symbolic links work, but they get tedious fast.

Every time you add a new config file, clone your dotfiles on a new machine, or reorganize your repository, you have to remember which links should exist and where they should point.

GNU Stow automates that.

Stow describes itself as a "symlink farm manager." In practical terms, it mirrors one directory structure into another directory by creating symbolic links back to the original files.

For dotfiles, that means:

- your real configuration files live in your dotfiles repo
- Stow creates links in your home directory
- programs keep reading config from the paths they already expect

One command can link the whole tree.

## Prerequisites

This tutorial assumes you already have a dotfiles directory directly inside your home directory:

```text
~/.dotfiles
```

Your folder might look like:

```text
~/.dotfiles/
  .profile
  .emacs.d/
  .config/
    polybar/
  README.org
  LICENSE
```

The exact files do not matter. What matters is that the layout inside `~/.dotfiles` matches where those files should appear relative to your home directory.

## Install GNU Stow

On most GNU/Linux distributions, the package is called `stow`.

For example:

```bash
sudo apt install stow
```

On macOS with Homebrew:

```bash
brew install stow
```

On Windows, you can install Stow through MSYS2:

```bash
pacman -S stow
```

For Windows, run the MSYS2 shell as administrator if you want Stow to create symbolic links correctly. You may also need to configure the MSYS environment so it creates links instead of copying files. If Stow silently copies files on Windows, that setting is usually the first place to check.

## Basic Usage

Go into your dotfiles directory:

```bash
cd ~/.dotfiles
```

Run:

```bash
stow .
```

That tells Stow to take the current directory and link its contents into the target directory.

Because `~/.dotfiles` is directly inside your home directory, Stow uses the parent directory as the default target:

```text
source: ~/.dotfiles
target: ~
```

So if your dotfiles repo contains:

```text
~/.dotfiles/.profile
~/.dotfiles/.emacs.d
~/.dotfiles/.config/polybar
```

Stow creates links like:

```text
~/.profile        -> ~/.dotfiles/.profile
~/.emacs.d        -> ~/.dotfiles/.emacs.d
~/.config/polybar -> ~/.dotfiles/.config/polybar
```

That is the whole magic trick. Nicely small, like a good shell command should be.

## How Stow Decides Where Links Go

Stow walks the source directory and recreates the same relative structure under the target directory.

If the source contains:

```text
.config/polybar/config
```

and the target is your home directory, Stow links it under:

```text
~/.config/polybar/config
```

This is why your dotfiles repo should mirror your home directory structure.

Good:

```text
~/.dotfiles/.config/polybar/config
```

Less convenient:

```text
~/.dotfiles/polybar/config
```

The mirrored structure gives Stow enough information to place links correctly.

## Existing Directories Are Handled Intelligently

Your home directory may already contain:

```text
~/.config
```

Stow does not need to replace that directory. It can place links inside it.

For example, if your repo contains:

```text
~/.dotfiles/.config/polybar
```

and your home directory already has:

```text
~/.config
```

Stow creates:

```text
~/.config/polybar -> ~/.dotfiles/.config/polybar
```

That behavior is exactly what you want for XDG-style config directories.

## Handling Conflicts

If a real file or directory already exists where Stow wants to create a link, Stow will complain.

For example, if this already exists:

```text
~/.profile
```

and Stow wants to create:

```text
~/.profile -> ~/.dotfiles/.profile
```

you will get a conflict.

Do not delete blindly. First inspect the existing file:

```bash
ls -al ~/.profile
```

If it contains configuration you want, move it into your dotfiles repo or back it up before removing it.

Once the path is clear, rerun:

```bash
cd ~/.dotfiles
stow .
```

Stow is safe to rerun. If a link already exists and points to the correct target, Stow leaves it alone.

## Use Explicit Source and Target Directories

The simple command works because `~/.dotfiles` lives directly under `~`:

```bash
cd ~/.dotfiles
stow .
```

If you keep dotfiles elsewhere, such as:

```text
~/src/dotfiles
```

you should specify the source and target explicitly:

```bash
stow --dir ~/src/dotfiles --target ~ .
```

Short form:

```bash
stow -d ~/src/dotfiles -t ~ .
```

If you use this often, put it in a small script so you do not have to remember the flags.

## Ignore Files and Directories

Stow ignores some common files by default, such as source-control folders and documentation files.

But your repo may contain other files that should not be linked into your home directory.

For example:

```text
README.org
notes.org
LICENSE
misc/
```

To control this, create:

```text
~/.dotfiles/.stow-local-ignore
```

Each line is a string or regular expression for a file or directory Stow should ignore.

Example:

```text
\.git
LICENSE
README.*
.*\.org
misc
```

Now rerun:

```bash
stow .
```

Ignored files will not be linked into your home directory.

## Important Ignore File Detail

When you create `.stow-local-ignore`, it replaces Stow's default ignore list.

That means files Stow previously ignored, such as `LICENSE`, may start getting linked unless you include them in your local ignore file.

So if you add a custom ignore file, include both:

- your project-specific ignores
- any default-style ignores you still care about

A practical starting point:

```text
\.git
\.gitignore
LICENSE
README.*
.*\.org
misc
```

Adjust it as your repo grows.

## Remove Stow Links

If you want to remove the links Stow created, use:

```bash
stow -D .
```

Run it from your dotfiles directory:

```bash
cd ~/.dotfiles
stow -D .
```

This removes the symbolic links from the target directory. It does not delete the real files in your dotfiles repo.

Afterward, paths such as:

```text
~/.profile
~/.emacs.d
~/.config/polybar
```

will no longer point into `~/.dotfiles`.

This is useful if you want to clean up links, test your setup, or temporarily disable a dotfiles package.

## Rerun Stow After Syncing

If your dotfiles are managed with Git, remember to run Stow after pulling changes on another machine.

For example:

```bash
cd ~/.dotfiles
git pull
stow .
```

This matters when the latest changes add new files. Git will fetch the files into your dotfiles repo, but Stow is what creates the links in your home directory.

If you forget this step, you may wonder why a new configuration file exists in the repo but the program cannot find it. The answer is usually: the file was never linked.

## A Simple Sync Script

You can automate the common flow with a small script.

For example, save this as:

```text
~/.dotfiles/bin/sync-dotfiles
```

```bash
#!/usr/bin/env bash

set -e

DOTFILES="$HOME/.dotfiles"
BRANCH="main"

cd "$DOTFILES"

git stash push -u -m "sync-dotfiles: local changes"
git pull origin "$BRANCH"

if git stash list | grep -q "sync-dotfiles: local changes"; then
  git stash pop || true
fi

if ! git diff --check; then
  echo "Resolve merge or whitespace conflicts before running stow."
  exit 1
fi

stow .
```

Make it executable:

```bash
chmod +x ~/.dotfiles/bin/sync-dotfiles
```

This script:

1. moves into your dotfiles repo
2. stashes uncommitted local changes
3. pulls from the remote branch
4. reapplies local changes
5. checks for obvious diff problems
6. runs `stow .`

If your primary branch is named `master`, change:

```bash
BRANCH="main"
```

to:

```bash
BRANCH="master"
```

This is only a starting point. Treat sync scripts with a little suspicion until you understand every command in them, especially anything involving `stash`.

## Where to Put Helper Scripts

If you create helper scripts, keep them in your dotfiles repo:

```text
~/.dotfiles/bin/sync-dotfiles
```

Then add that directory to your shell `PATH`:

```bash
export PATH="$HOME/.dotfiles/bin:$PATH"
```

Now you can run:

```bash
sync-dotfiles
```

from anywhere.

## Recommended Workflow

For a simple Stow-based dotfiles setup:

1. Put your dotfiles in `~/.dotfiles`.
2. Mirror the home directory layout inside the repo.
3. Add `.stow-local-ignore`.
4. Run `stow .`.
5. Commit the repo to Git.
6. After each `git pull`, run `stow .` again.

If you need to remove links:

```bash
stow -D .
```

If you need to recreate them:

```bash
stow .
```

Stow is useful because it does not try to own your configuration. It just creates the links you would have created by hand, consistently and repeatably.
