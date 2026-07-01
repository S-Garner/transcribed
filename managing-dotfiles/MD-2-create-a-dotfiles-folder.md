# Managing Dotfiles 2: Create a Dotfiles Folder

Before you can publish, sync, or automate your dotfiles, you need a place to keep them.

The basic idea is:

1. Create a `dotfiles` folder.
2. Move selected configuration files into it.
3. Link those files back to the places where programs expect them.
4. Verify that the programs still load their configuration correctly.

This tutorial walks through that manual process. Later, tools like GNU Stow can automate most of it, but doing it by hand once makes the model much clearer.

## Create the Folder

Create a `dotfiles` directory directly inside your home directory:

```bash
mkdir ~/dotfiles
```

Keeping it at the root of your home directory is simple and works well with dotfile-management tools. You can put it elsewhere, but many tools and examples assume a layout like this:

```text
~
  dotfiles/
  .bashrc
  .emacs.d/
  .config/
```

The `dotfiles` folder will become the canonical home for the configuration files you choose to manage.

## Choose Files to Manage

Start with configuration you actually care about.

Good first candidates:

- your editor config, such as `~/.emacs.d`
- your shell config, such as `~/.bashrc`
- selected app config under `~/.config`

Avoid moving everything blindly.

For example, `~/.bash_history` is usually not a good dotfile candidate. It is machine-local shell history, not intentional configuration.

Likewise, `~/.config` may contain many files created by your desktop environment or distribution. Some are worth keeping, but many are generated state or machine-specific settings.

Start small. You can always add more later.

## Mirror the Home Directory Layout

When possible, keep the same structure inside `~/dotfiles` that the files had in your home directory.

For example, if an app expects:

```text
~/.config/mpv/mpv.conf
```

store it in your dotfiles repo as:

```text
~/dotfiles/.config/mpv/mpv.conf
```

That mirrored layout makes linking easier and works especially well with tools like GNU Stow.

## Move an Emacs Configuration

If you have an Emacs configuration at:

```text
~/.emacs.d
```

move it into your dotfiles folder:

```bash
mv ~/.emacs.d ~/dotfiles/.emacs.d
```

Now the real files live here:

```text
~/dotfiles/.emacs.d
```

But Emacs still expects to find them at:

```text
~/.emacs.d
```

You will fix that with a symbolic link.

## Move a Bash Configuration

Move your Bash config:

```bash
mv ~/.bashrc ~/dotfiles/.bashrc
```

Do not move shell history:

```text
~/.bash_history
```

That file changes constantly and is usually specific to one machine.

You may also have files such as:

```text
~/.bash_profile
~/.profile
~/.bash_logout
```

Inspect them before moving. Some distribution-provided files may contain system-specific setup you do not actually want to sync.

## Move an App Config Under `.config`

Suppose you use `mpv`, and its config lives at:

```text
~/.config/mpv
```

First create the matching `.config` directory inside your dotfiles repo:

```bash
mkdir -p ~/dotfiles/.config
```

Then move the app folder:

```bash
mv ~/.config/mpv ~/dotfiles/.config/mpv
```

Now your dotfiles folder might look like:

```text
~/dotfiles/
  .bashrc
  .emacs.d/
  .config/
    mpv/
```

This keeps the structure close to the real home directory layout.

## Create Symbolic Links

After moving the files, the original paths no longer exist. Programs will not know where to find their configuration.

Symbolic links solve that.

A symbolic link is a pointer from the expected location to the real file location.

The command on Linux and macOS is:

```bash
ln -sf TARGET LINK_NAME
```

The target comes first. The link location comes second.

That order is easy to forget. Think: "link this real thing here."

## Link `.emacs.d`

Create a link from `~/.emacs.d` to the real directory in `~/dotfiles`:

```bash
ln -sf ~/dotfiles/.emacs.d ~/.emacs.d
```

Verify it:

```bash
ls -al ~/.emacs.d
```

You should see that `~/.emacs.d` points to `~/dotfiles/.emacs.d`.

Then start Emacs and make sure your configuration loads correctly.

## Watch for Recreated Directories

There is a small trap here.

If a program recreates the original directory after you move it, then `ln -s` may create the symlink inside that directory instead of replacing the directory.

For example, if `~/.emacs.d` already exists as a directory, this command may create:

```text
~/.emacs.d/.emacs.d -> ~/dotfiles/.emacs.d
```

That is not what you want.

Check first:

```bash
ls -al ~/.emacs.d
```

If the old directory was recreated and you are sure it contains nothing important, remove it:

```bash
rm -r ~/.emacs.d
```

Then create the link again:

```bash
ln -sf ~/dotfiles/.emacs.d ~/.emacs.d
```

Be careful with `rm -r`. Inspect before deleting. The filesystem does not do dramatic music before bad decisions, which is rude but consistent.

## Link `.bashrc`

Create the Bash link:

```bash
ln -sf ~/dotfiles/.bashrc ~/.bashrc
```

Open a new terminal or source the file:

```bash
source ~/.bashrc
```

If your prompt, aliases, environment variables, or shell functions still work, the link is doing its job.

## Link `.config/mpv`

For an app directory under `.config`, link the folder back into `~/.config`:

```bash
ln -sf ~/dotfiles/.config/mpv ~/.config/mpv
```

Verify:

```bash
ls -al ~/.config/mpv
```

It should point to:

```text
~/dotfiles/.config/mpv
```

Then run the app and confirm it still reads the configuration.

## Full Manual Example

Here is the full flow for the example files:

```bash
mkdir ~/dotfiles

mv ~/.emacs.d ~/dotfiles/.emacs.d
mv ~/.bashrc ~/dotfiles/.bashrc

mkdir -p ~/dotfiles/.config
mv ~/.config/mpv ~/dotfiles/.config/mpv

ln -sf ~/dotfiles/.emacs.d ~/.emacs.d
ln -sf ~/dotfiles/.bashrc ~/.bashrc
ln -sf ~/dotfiles/.config/mpv ~/.config/mpv
```

Then verify each tool:

```bash
emacs
source ~/.bashrc
mpv some-video-file
```

The details will differ for your setup, but the pattern stays the same:

1. Move the real config into `~/dotfiles`.
2. Link from the original location back to the moved config.
3. Test the program.

## Windows: Use `mklink`

Windows does not use `ln -s`.

In Command Prompt, use `mklink`.

For individual files:

```cmd
mklink /H LINK_NAME TARGET
```

For directories:

```cmd
mklink /J LINK_NAME TARGET
```

The order is different from `ln`.

With `ln`, the target comes first:

```bash
ln -sf TARGET LINK_NAME
```

With `mklink`, the link name comes first:

```cmd
mklink /J LINK_NAME TARGET
```

You usually need an elevated Command Prompt to create these links. Open the Start menu, search for `cmd`, right-click Command Prompt, and choose "Run as administrator."

Windows dotfile management can be less consistent because many programs store configuration in the Registry or under `AppData`, but tools that use normal files can still be managed this way.

## The Downside of Manual Symlinks

Manual symbolic links work, but they get annoying.

They are fine on the first machine:

1. Move files.
2. Create links.
3. Test everything.

The annoyance appears on the second machine. After cloning your dotfiles repo, you have to remember every link you created and recreate them all correctly.

You can write your own setup script, but then the script becomes another thing to maintain.

That is why dotfile tools exist.

GNU Stow, for example, can create and manage these symbolic links for you based on your folder structure. Other tools can do similar work.

Still, learning the manual symlink version is useful because it explains what those tools are doing under the hood.

## Recommended Starting Point

Start with a small set of files:

```text
~/dotfiles/
  .bashrc
  .emacs.d/
  .config/
    mpv/
```

Create links manually once.

Verify everything works.

Then put the folder under Git in the next step.

This gives you a simple dotfiles folder that is ready to become a real repository.
