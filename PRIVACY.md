# Privacy

HourLog runs on your PC and keeps your data there.

## What HourLog stores

- Your time entries, projects, clients, tags, plans, and settings, in
  `%ProgramData%\HourLog\hourlog.db`.
- Daily backups of that file, in the backup folder that you pick in **Settings > Backup**.
- The position and options of the timer widget, in `%LocalAppData%\HourLog\widget.json`.

HourLog has no account, no telemetry, and no analytics.

## What goes online

HourLog makes one kind of network call: the update check.

- Every 6 hours, HourLog asks the GitHub API for the latest release of
  [ilievici/HourLog](https://github.com/ilievici/HourLog/releases).
- When a newer version exists, HourLog downloads its installer from GitHub.

The request carries the HourLog version in its `User-Agent` header. It carries none of your data.
GitHub sees your IP address, as with any website. See the
[GitHub privacy statement](https://docs.github.com/site-policy/privacy-policies/github-general-privacy-statement).

To turn the update check off, open **Settings > Updates** and turn off **Check for updates**.
HourLog then makes no network calls at all.

## Other PCs

HourLog listens only on your own PC (`localhost`). Other PCs on your network can't open it.
