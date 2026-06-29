# Emacs From Scratch 11: Keeping Packages Up to Date

Once your Emacs configuration starts using packages from ELPA, MELPA, Org, or other archives, you eventually need to update them.

Emacs will not remind you automatically. It does not check in the background, it does not show a badge, and it will not quietly upgrade packages unless you configure something to do that. Package updates are something you initiate yourself.

This tutorial covers two approaches:

- manually updating packages with Emacs' built-in package menu
- prompting for updates automatically with `auto-package-update`

The manual workflow is simple and is worth understanding even if you later automate it.

## Open the Package List

Emacs includes a package management UI called the package menu.

Open it with:

```text
M-x list-packages
```

When the package list opens, Emacs refreshes the configured package archives. In the Emacs From Scratch configuration, those archives include sources such as GNU ELPA, MELPA, and the Org package archive.

After the refresh finishes, Emacs shows a buffer containing known packages, installed packages, available versions, and package descriptions.

If updates are available, the echo area will say something like:

```text
Packages that can be upgraded: 3; type `U' to mark for upgrading
```

That message is easy to miss, but it is the key to the update workflow.

## Mark Packages for Upgrade

In the package list buffer, press:

```text
U
```

That marks all upgradeable packages for installation.

This does not upgrade them immediately. Like Dired, the package menu often works in two phases:

1. Mark the things you want to change.
2. Execute the marked actions.

After pressing `U`, execute the upgrades with:

```text
x
```

Emacs will show the packages it plans to upgrade and ask for confirmation.

Confirm with:

```text
yes
```

Emacs then downloads the newer package versions, installs them into your local package directory, and byte-compiles them when necessary.

## Restart After Updating

After package upgrades, restart Emacs.

In some cases, Emacs can use the new package version without a restart, especially if the package has not been loaded yet. In practice, restarting is the boring-but-sane option. It avoids mixed states where part of your session is using old code and another part might load new code.

Exit Emacs with:

```text
C-x C-c
```

Then start Emacs again.

## Verify Packages Are Up to Date

After restarting, open the package list again:

```text
M-x list-packages
```

Then press:

```text
U
```

If everything is current, Emacs will report:

```text
No packages to upgrade
```

That confirms the update process completed.

## Automate Package Update Checks

If you do not want to remember to run `list-packages` yourself, you can use the `auto-package-update` package.

The goal is not necessarily to update silently. A good setup is to let Emacs check on a schedule and ask whether you want to update now.

Add this near the package setup section of your configuration:

```elisp
(use-package auto-package-update
  :custom
  (auto-package-update-interval 7)
  (auto-package-update-prompt-before-update t)
  (auto-package-update-hide-results nil)
  :config
  (auto-package-update-maybe)
  (auto-package-update-at-time "09:00"))
```

This installs `auto-package-update` and configures three important variables.

## Set the Update Interval

This option controls how many days must pass between automatic update checks:

```elisp
(auto-package-update-interval 7)
```

With `7`, Emacs checks once per week.

You can choose whatever interval fits your tolerance for change:

```elisp
(auto-package-update-interval 1)   ;; daily
(auto-package-update-interval 14)  ;; every two weeks
(auto-package-update-interval 30)  ;; roughly monthly
```

The package records when it last performed an update. Future checks compare the current date against that saved timestamp.

## Prompt Before Updating

This option makes Emacs ask before installing updates:

```elisp
(auto-package-update-prompt-before-update t)
```

That is usually safer than silent updates.

Package updates can occasionally break a configuration, so it is useful to have a chance to say no when you are about to start work, give a presentation, or do anything where Emacs needs to stay boring and functional.

With this enabled, Emacs will ask:

```text
Auto update packages now?
```

You can answer yes or no.

## Show or Hide Update Results

This option controls whether Emacs shows the update results buffer:

```elisp
(auto-package-update-hide-results nil)
```

With `nil`, Emacs shows which packages were updated.

If you do not want to see that buffer, set it to `t`:

```elisp
(auto-package-update-hide-results t)
```

Showing the results is useful while you are still learning your setup. If something breaks after an update, you have a better clue about what changed.

## Check at Startup

This line checks whether the configured interval has passed:

```elisp
(auto-package-update-maybe)
```

If enough days have passed, it starts the update process. If prompting is enabled, it asks first.

Putting this in your configuration means Emacs checks during startup.

That works well if you start a fresh Emacs every day.

## Check at a Scheduled Time

Some people keep Emacs open for days or weeks. For that workflow, startup checks are not enough.

This line schedules an update check at 9:00 AM:

```elisp
(auto-package-update-at-time "09:00")
```

The time uses 24-hour format.

Together, these two lines cover both habits:

```elisp
(auto-package-update-maybe)
(auto-package-update-at-time "09:00")
```

If you restart Emacs often, the startup check catches updates. If you leave Emacs running, the scheduled check catches them.

## Update on Demand

You do not have to enable automatic checking.

If you prefer manual control, you can install `auto-package-update` but skip the startup and scheduled checks. Then run this command whenever you want to update:

```text
M-x auto-package-update-now
```

That updates immediately, regardless of the interval.

It also resets the package's last-update timestamp, so the next automatic prompt is calculated from that time.

## Should You Update Automatically?

Automatic package updates are convenient, but they have risk.

Many MELPA packages are built directly from upstream Git commits. That is one of MELPA's strengths: you can get improvements quickly. It also means you can occasionally receive a bug, a removed function, a renamed variable, or a changed interface right after the package maintainer commits it.

That does not mean you should avoid updates. It means you should update deliberately.

Good habits:

- Prompt before updating instead of updating silently.
- Avoid updating right before important work.
- Restart Emacs after updating.
- Keep your Emacs configuration in Git.
- Commit a known-good configuration before large updates.

If something breaks, Git gives you a way to inspect what changed and roll back your own configuration changes. Package rollback is more nuanced with `package.el`, but knowing which packages changed still helps.

## Backing Up Packages

`auto-package-update` provides hooks such as:

```elisp
auto-package-update-before-hook
```

You can use that hook to back up your package directory before updating.

That can be useful if you want a quick escape hatch when an update breaks something. The idea is to copy your current package directory before the update starts, then restore it if needed.

The exact backup code depends on where your package directory lives and how you want to manage backups, so treat this as an option rather than a required part of the basic setup.

## A Practical Configuration

This is a reasonable default if you want weekly reminders but not silent updates:

```elisp
(use-package auto-package-update
  :custom
  (auto-package-update-interval 7)
  (auto-package-update-prompt-before-update t)
  (auto-package-update-hide-results nil)
  :config
  (auto-package-update-maybe)
  (auto-package-update-at-time "09:00"))
```

If you want full manual control, use this instead:

```elisp
(use-package auto-package-update
  :commands (auto-package-update-now))
```

Then update only when you choose:

```text
M-x auto-package-update-now
```

## Recommended Workflow

For most Emacs configurations, a steady workflow looks like this:

1. Keep your configuration in Git.
2. Let Emacs prompt before package updates.
3. Update when you have a little time to fix issues.
4. Restart Emacs after the update.
5. Run `M-x list-packages`, press `U`, and confirm there are no packages left to upgrade.

Package maintenance does not need to be scary. The trick is to avoid surprise. Emacs will happily let you control when updates happen, and with a small amount of configuration, it can also remind you when it is time.
