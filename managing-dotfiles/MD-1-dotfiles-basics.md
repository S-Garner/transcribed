# Managing Dotfiles 1: Dotfiles Basics

Dotfiles are your personal configuration.

They are the text files, folders, and scripts that shape how your tools behave: your shell, editor, Git setup, terminal, window manager, and many other programs. If you have spent time making your computer feel like yours, that work probably lives in dotfiles.

This tutorial explains what dotfiles are, where they usually live, why people keep them in source control, and why publishing them can be useful.

## What Are Dotfiles?

Dotfiles are usually text-based configuration files stored in your home directory.

They get their name from the leading dot in filenames such as:

```text
.bashrc
.bash_profile
.vimrc
.emacs.d
.config
```

On Unix-like systems, files and directories that start with `.` are hidden by default. That keeps configuration files from cluttering normal directory listings while still leaving them editable.

The important idea is not the dot itself. The important idea is that these files define your working environment.

Examples include:

- shell configuration
- editor configuration
- Git configuration
- terminal configuration
- window manager configuration
- application settings
- personal scripts

If a file helps recreate the way you use your machine, it is a good candidate for your dotfiles.

## Text Configuration Matters

Many Linux and macOS programs use plain text configuration files. That is valuable because text files are easy to:

- edit directly
- compare over time
- search
- copy between machines
- store in Git
- publish and share

Not every program works this way. Some applications use binary configuration formats, databases, registries, or internal settings stores. Those are harder to inspect and harder to manage as part of a personal configuration repository.

The most pleasant tools tend to expose configuration as text. Text gives you ownership.

## Common Dotfiles

Shells usually have dotfiles in your home directory.

For Bash, common files include:

```text
~/.bash_profile
~/.bashrc
```

`~/.bash_profile` is often used for login shell setup.

`~/.bashrc` is often used for interactive shell setup when you open a new terminal.

Editors also commonly use dotfiles:

```text
~/.emacs.d
~/.vimrc
~/.vim
```

Git has its own user-level configuration:

```text
~/.gitconfig
```

That file can store your name, email address, aliases, default branch behavior, editor preference, and other Git settings.

## The `.config` Directory

Modern Linux applications often store configuration under:

```text
~/.config
```

You might see paths such as:

```text
~/.config/i3
~/.config/alacritty
~/.config/fish
~/.config/nvim
~/.config/mpv
```

This convention comes from the XDG base directory specification. Instead of scattering every program's configuration directly in your home directory, applications can place their settings under a shared configuration directory.

This makes dotfiles easier to organize, especially when many programs are involved.

## Dotfiles on Windows

Dotfiles are less central on Windows because Windows programs often store configuration in places such as:

- the Registry
- application-specific folders
- `Program Files`
- `%APPDATA%`

Still, many cross-platform developer tools use text configuration on Windows too.

Common locations include:

```text
C:\Users\<you>
C:\Users\<you>\AppData\Roaming
```

If you use tools like Git, Emacs, Vim, Neovim, VS Code, PowerShell, or cross-platform terminal applications, you may still have useful configuration files worth tracking.

## Personal Scripts Count Too

Dotfiles are not only configuration files.

Personal scripts belong there too.

For example, you might have scripts that:

- start a development environment
- clean up files
- run backups
- manage displays
- automate repetitive project tasks
- wrap command-line tools with your preferred defaults

If a script is part of your daily workflow and you would miss it on a new machine, treat it as part of your dotfiles.

## Why Manage Dotfiles?

The moment you customize more than one or two tools, your configuration becomes real work.

Managing dotfiles gives that work a home.

Instead of leaving important configuration scattered across one machine, you can collect it in one repository and preserve it.

The usual pattern is:

1. Create a dotfiles directory, such as `~/dotfiles`.
2. Move or copy your configuration files into it.
3. Put that directory under source control.
4. Link the files back into the locations where programs expect them.

Future tutorials in this series can go deeper into linking strategies, but the core idea is simple: keep the canonical version in your dotfiles repo, then place links where applications look for config.

## Use Source Control

The best way to manage dotfiles is with source control.

Git is the most common choice:

```bash
mkdir ~/dotfiles
cd ~/dotfiles
git init
```

You can use other decentralized version control systems, such as Mercurial or Darcs, but Git has the strongest ecosystem and the most hosting options.

Git works well for dotfiles because a repository can live entirely on your local machine. You do not need a server to start. Later, if you want to sync between machines, you can push the repository to a host such as GitHub, GitLab, or another Git server.

## Benefits of a Dotfiles Repository

A dotfiles repository makes your setup portable.

If you use multiple machines, you can sync the same configuration across them. That means your shell, editor, Git aliases, scripts, and tool preferences can follow you.

It also gives you history.

When you change a configuration and later regret it, Git can show you what changed and help you roll back.

It gives you room to experiment.

If you want to try a new shell, editor layout, window manager, or Emacs setup, create a branch:

```bash
git checkout -b try-new-shell
```

Make the risky changes there. If they work, merge them. If they do not, switch back to your main branch.

Most importantly, a repository protects the time you spent crafting your environment. Your setup stops being a fragile pile of files and becomes a project you can carry forward.

## Should You Publish Dotfiles?

Publishing dotfiles can feel strange at first. Your configuration is personal. It reflects your habits, preferences, and maybe a little chaos from years of experimentation.

Still, publishing can be valuable.

First, it makes your setup easier to sync and recover. If your machine dies, your configuration is still available.

Second, it helps other people learn. Dotfiles repositories are full of practical examples: shell tricks, editor setups, useful scripts, package choices, and workflow ideas.

Third, it lets you learn from others. Browsing other people's dotfiles is one of the best ways to discover tools and configuration patterns you would not have found otherwise.

## Be Careful Before Publishing

Before making a dotfiles repository public, check it carefully.

Do not publish secrets.

Watch for:

- API keys
- SSH private keys
- access tokens
- passwords
- private hostnames
- work-specific credentials
- personal email addresses if you do not want them public
- machine-specific paths that reveal information you care about

Use `.gitignore` aggressively for generated files, caches, secrets, and local-only configuration.

If a program mixes public configuration and private secrets in one file, consider splitting it if the tool supports includes. For example, keep safe defaults in the public repo and load private local settings from an ignored file.

## Literate Dotfiles

If you use Emacs, you can take dotfiles further with literate configuration.

In a literate setup, your configuration lives inside an Org Mode document. You write prose explaining what each section does, then include source blocks containing the actual configuration.

That gives you a file that is both:

- documentation
- executable configuration

For example, an `emacs.org` file can explain your editor setup while also generating the `init.el` file Emacs loads.

This style is especially nice for public dotfiles because readers can understand not only what you configured, but why.

## What Belongs in a Dotfiles Repo?

Good candidates:

- `.bashrc`
- `.bash_profile`
- `.zshrc`
- `.gitconfig`
- `.emacs.d` or Emacs literate config
- `.vimrc`
- `.config` subdirectories for tools you actively configure
- terminal emulator settings
- window manager settings
- personal scripts
- package lists
- setup notes

Usually avoid:

- caches
- logs
- compiled output
- secrets
- machine-generated state
- large binary files
- temporary files

The goal is to preserve intentional configuration, not every file a program happens to create.

## A Good Starting Structure

A simple dotfiles repository might start like this:

```text
dotfiles/
  README.md
  bash/
    .bashrc
    .bash_profile
  git/
    .gitconfig
  emacs/
    .emacs.d/
  scripts/
    bin/
      project-clean
      start-dev
```

There is no one correct structure. Choose something you can understand six months later.

As your setup grows, you can adopt tools like GNU Stow or a custom installation script to link files into the right places.

## Why This Is Worth Doing

Dotfiles are easy to ignore until you lose them.

A good dotfiles repo gives you:

- repeatable setup
- history
- rollback
- experimentation through branches
- syncing between machines
- a place to document your workflow
- a useful artifact to share with others

Your configuration is part of your craft. Treating it like a project makes it easier to improve without fear.

## Next Steps

Once you understand the idea, the next step is practical dotfile management:

- create a dotfiles repository
- move configuration files into it
- link them back into your home directory
- publish the repository
- use tools like GNU Stow to manage links cleanly

That is where dotfiles stop being just hidden files and start becoming a portable, personal system.
