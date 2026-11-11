# HourLog

HourLog is a personal time tracker for Windows. It runs on your PC, opens in its own app window, and
keeps all your data in one local file. It works without internet and needs no account.

## Download

1. Open the [latest release](https://github.com/ilievici/HourLog/releases/latest).
2. Under **Assets**, download `HourLog-Setup-<version>.exe`.
3. Run it and accept the admin prompt.

The installer isn't signed, so SmartScreen might warn you. Click **More info**, then **Run anyway**.
Then open **HourLog** from the Start menu.

You need Windows 10 version 1809 or later, 64-bit.

## Features

- Start a timer or add time by hand, from the app or the always-on-top widget.
- Organize time by client, project, task, tag, and your own custom fields.
- Fill in a weekly timesheet and export booked hours with rounding.
- Plan the coming days and compare planned with tracked time.
- See reports for any period and export them to CSV, Excel, and PDF.
- Get reminders when no timer runs, when a timer runs long, and at the end of the day.
- Keep your data safe with daily backups and a full export you can import on another PC.
- Use light, dark, or system themes.

## Updates

HourLog checks this page for new versions every 6 hours. When it finds one, it downloads the
installer in the background and offers an **Update** button. Your data and settings stay as they
are, and HourLog backs up the database first.

To keep HourLog fully offline, turn off **Check for updates** in **Settings > Updates**. To update by
hand, download the new installer and run it.

## Your data

HourLog stores everything in `%ProgramData%\HourLog\hourlog.db` on your PC. Nothing is sent
anywhere. See [Privacy](PRIVACY.md) for the one network call HourLog makes.

To back up HourLog, copy that file, or use **Settings > Backup**. Uninstalling HourLog keeps your
data.

## Documentation

- [User guide](docs/user-guide.md): each page of the app, the widget, and the settings.
- [API reference](docs/api.md): the local HTTP API for your own scripts.

## License

HourLog is freeware for personal use. See the [license](LICENSE.md).

## Support

To report a problem or ask for a feature, open an
[issue](https://github.com/ilievici/HourLog/issues). Add your HourLog version from
**Settings > About HourLog**.
