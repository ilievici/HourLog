# HourLog user guide

HourLog tracks your working time on one Windows PC. It runs as a Windows service on
`http://localhost:5280` and stores everything in one SQLite file. It works without internet.

This guide covers each page of the app, the timer widget, and the settings. For the HTTP API, see
[api.md](api.md).

## Install and upgrade

1. Open the [latest release](https://github.com/ilievici/HourLog/releases/latest).
2. Under **Assets**, download `HourLog-Setup-<version>.exe`.
3. Run `HourLog-Setup-<version>.exe` and accept the admin prompt.

The installer isn't signed, so SmartScreen might warn you. Click **More info**, then
**Run anyway**.

The installer sets up the following:

- The `HourLog` Windows service in `C:\Program Files\HourLog`. It starts with Windows and restarts
  after a crash.
- A Start menu entry named **HourLog**. It opens the app in its own Edge window, or in Chrome if
  Edge is missing. You can pin it to the taskbar.
- A Start menu entry named **HourLog Timer Widget** for the always-on-top timer. The installer can
  also start the widget when you sign in to Windows.
- An uninstaller in **Settings > Apps**.

To upgrade, run a newer installer. It keeps your database and your `appsettings.json`.
Uninstalling also keeps your data in `%ProgramData%\HourLog`.

### Update from the app

HourLog checks GitHub for a new version one minute after it starts and then every 6 hours. When it
finds one, it downloads the installer in the background and checks its SHA-256 checksum. Then the
following happens:

- The tray icon shows a notification.
- A bar under the header says that the new version is ready to install. Click the close button
  to hide the bar for that version. You can still install it from **Settings > Updates**.

To install it, click **Update**, then **Update now**. HourLog does the following:

1. Backs up the database.
2. Runs the installer silently. The app closes for about a minute.
3. Starts the service and the timer widget again. The page reloads with the new version.

A running timer keeps running. Your data and `appsettings.json` stay as they are. If the install
fails, the service starts again with the old version. The installer log is
`%ProgramData%\HourLog\updates\install.log`.

The update check is the only time HourLog goes online. To keep HourLog fully offline, turn off
**Check for updates** in **Settings > Updates**. You can still click **Check now** there.

## Track time

The **Tracker** page shows the timer bar and this week's entries grouped by day. The header shows
your progress against the daily and weekly targets on every page.

### Use the timer

1. Type what you work on in **What are you working on?**. Suggestions from earlier entries appear
   as you type. Use the arrow keys to pick one.
2. Optional: set a project, task, and tags.
3. Press `Enter` or click **Start**.

To change when the running timer started, click its elapsed time. Type the new time in **Start time**
and press `Enter`. The time keeps the day the timer started on. A start in the future is refused.
You can undo the change from the message that appears.

To stop the timer, press `S` or click **Stop**. If you turned on rounding, a dialog shows the
rounded time first. Nothing changes until you confirm.

HourLog saves hours and minutes, not seconds. The running timer shows seconds so you can see it
count. When you stop it, the entry drops the seconds of its start and end. A timer that you stop
within the same minute adds no entry.

To restart earlier work, click **Continue** on its entry. A new timer starts with the same
description, project, task, and tags.

### Add time by hand

Press `M` or click the mode switch to add an entry with a start and an end time. Durations accept
`1.5`, `1,5`, `1:30`, `90m`, `1h30`, and `1h 30m`. A bare number means hours. HourLog rounds a
duration to the nearest minute.

An entry can end after midnight. HourLog shows a marker on the end time when an entry ends on the
next day.

### Edit entries

- Click a description, project, time, or date to change it in place.
- Open the **More actions** menu on an entry to round it up, split it, duplicate it, edit its
  custom fields, or delete it.
- Right-click the project of an entry, or press Shift+F10 on it, for two more options:
  - **Go to project** opens the project's edit dialog on the Projects page.
  - **Copy** puts one value of the entry on the clipboard: the project, the duration, or a custom
    field like a booking code. **All fields** copies every value, one per line.
  The **More actions** menu has the same two options.
- Select several entries with their checkboxes to change or delete them together.
- Each change shows a toast with **Undo**. Undo reverts the whole change, including bulk edits and
  imports.

### Keyboard shortcuts

| Key | Action |
| --- | --- |
| `N` or `/` | Focus **What are you working on?** |
| `Enter` | Start the timer, or add the entry in manual mode |
| `S` | Start or stop the timer |
| `M` | Switch between timer and manual mode |

On the **Planner** page, `N` and `/` focus the quick-add field.

### Go back to the top

On a long page, such as **Settings**, **Projects**, or **Timesheet**, a button with an up arrow appears in the
bottom-left corner after you scroll down. Click it, or press `Enter` when it has focus, to go back to the top.
Focus then moves to the page tab, so you can go on with the keyboard.

### When your PC sleeps

If your PC sleeps while a timer runs, HourLog asks what to do with that time when you come back:

- **Remove it and continue from** the wake-up time. HourLog stops the entry when the PC went to
  sleep and starts a new timer at wake-up.
- **Remove it and stop the timer** at the time the PC went to sleep.
- **Keep all** the time.

## Fill in a weekly timesheet

The **Timesheet** page shows a week as a grid. Each row is a project and task. Each column is a
day.

- To add time, type a duration in a cell, such as `1:30`, `1.5`, or `90m`, then press `Enter`.
  HourLog adds an entry. It copies the description, tags, and custom fields from the latest entry
  in that row.
- To change a cell that already has entries, type the new total. HourLog asks how to apply the
  difference:
  - **Add a new entry** with the extra time.
  - **Make the latest entry** longer or shorter.
  - **Replace** all entries in the cell with one entry.
- To clear a cell, type `0`. HourLog asks before it deletes the entries.
- To add a row, pick a project and task in **Add row**.
- To start a new week with the same rows, click **Copy last week's rows**.

You can't change a cell while its timer runs. Stop the timer first.

## Book your hours

Booking means that you export a week of rounded hours and enter them in the system where your
company records time. After you book a week, HourLog locks its entries so that the booked totals
can't drift.

### Export a week for booking

1. On the **Timesheet** page, open the week.
2. Click **Export for booking**. HourLog checks every entry in the week.
3. Review the result:
   - If every entry has a project and its required fields, download the CSV.
   - If some entries need a fix, fix them on the **Tracker** page, or click
     **Leave them out and download** and book them later.
4. Enter the hours in your booking system.
5. Click **Mark as booked**. HourLog locks the week.

The CSV has one row per date, project, and task. Each row contains:

- The client, project, and task.
- The custom fields of the project and the task, such as a booking code.
- The hours rounded to the step that you set in **Settings > Booking**, and the exact hours.
- A memo that lists what you worked on, with the rounded hours split across the activities. An
  activity is the first Jira key in a description, or the description itself.

You can also export the booking CSV for any period from the **Analytics** page. Open the export
menu and choose **For booking, rounded with a memo**.

### Locked weeks

Entries on a booked day can't be added, changed, moved, or deleted. Imports skip rows on booked
days. The **Tracker** page marks booked days.

To change a booked week, open it on the **Timesheet** page and click **Unlock week**.

The **Timesheet** tab shows a badge when a finished week isn't booked yet. A banner on the page
lists those weeks so that you can open each one.

## Plan your work

The **Planner** page shows the coming days. Each day lists plan items with an estimate. Time that
you track counts toward the matching item:

- If the item title has a Jira key, entries with the same key count.
- Otherwise, entries with the same text or on the same task count.

From a plan item, you can start a timer, mark it done, or drag it to another day. When past days
have unfinished items, click **Move N unfinished items to today** to move them.

To clone a plan item, open its **...** menu:

- **Duplicate** adds a copy right below the item.
- **Copy to today** and **Copy to tomorrow** add a copy to that day.

You can also hold **Ctrl** while you drop a dragged item. The item stays where it was, and a copy
goes where you drop it. A copy keeps the title, estimate, project, task, and notes, and starts not
done. To remove the copy, click **Undo** in the message.

To import meetings from Outlook, save your calendar as an `.ics` file. In Outlook, click
**File > Save Calendar**. Then open the **...** menu on the **Planner** page and click
**Import calendar**. HourLog reads the file offline. It doesn't sign in to Outlook.

The **Tracker** page shows **Today's plan** when the planner is on. To turn the planner off, go to
**Settings > Modules**. Turning it off hides the page and keeps its data.

To hide the items of **Today's plan**, click its title or the arrow next to it. The summary and
**Open planner** stay visible. HourLog remembers your choice in this browser.

## Keep a journal

The **Journal** page holds one note per day about how the day went.

1. Type a few lines in **Today's note**. To write about another day, pick it in the date box.
2. Optional: under **Mood**, pick **Positive**, **Neutral**, or **Negative**. **Auto** lets the
   analysis decide.
3. Click **Save**, or press **Ctrl+Enter**. The note then shows as saved, with the time. Click
   **Edit** to change it. Saving again on the same day updates the same note.

Click **Analyze** to see how the day went:

- The mood and a score from -1 (very negative) to +1 (very positive).
- The words that moved the score. A negation flips a word, so "not bad" counts as positive.
- The work you did, such as "Finished the report" or "Am trimis oferta".
- The work you didn't do, such as "Didn't call the client" or "N-am apucat să scriu testele".

Lines that start with `[x]` or `✓` count as done. Lines that start with `[ ]` or `✗` count as not
done. The analysis reads English and Romanian, with or without diacritics. It runs on your PC and
sends nothing to the internet. It uses word lists, so it can be wrong. When it is, pick the mood
yourself. Your choice shows as **(set by you)** and counts in the chart.

### Rewrite a note

**Rewrite** improves the writing, fixes the grammar, or translates the note to English. Use it
when a note mixes Romanian and English and you want it in one language.

1. Turn the writing assistant on in **Settings > Writing assistant**. Pick **Claude** and paste
   your Anthropic API key, or pick **Ollama** for a model that runs on your PC.
2. Optional: select the part of the note to rewrite. Without a selection, the whole note is
   rewritten.
3. Click **Rewrite** and pick **Improve writing**, **Fix grammar**, or **Translate to English**.
   You can also right-click in the text box.
4. Check the suggestion next to your text. You can edit it.
5. Click **Replace**. Your note changes only now, and **Undo** brings the old text back. Click
   **Save** to keep it.

Shift + right-click opens the browser menu with its spelling suggestions. The assistant is off by
default. With Claude, only the text you send to rewrite leaves your PC. HourLog saves the API key
encrypted and leaves it out of the full export. **Analyze** always works offline.

**Mood over time** shows one bar per note for the period you pick: up for positive, down for
negative. Above the chart you see the average mood, the days per mood, and the done and not done
totals.

**All notes** lists the notes of the period, newest first. Search ignores case and diacritics.
Click a note to read, change, analyze, or delete it. Each save and delete shows **Undo**.

To turn the journal off, go to **Settings > Modules**. Turning it off hides the page and keeps your
notes. Notes are in the daily backups and the full export.

## Analyze your time

The **Analytics** page shows where your hours went for any period, from this week to the last 12
months, or for custom dates. Filter by client, project, task, tag, booked status, or text. The
task filter appears after you pick a project.

The page has three views:

- **Overview**: charts and key figures.
- **Summary**: totals grouped by up to three levels, such as client, then project, then task. You
  can group by client, project, task, tag, Jira key, description, or date. Click a row to open the
  level below it. An entry with several tags counts once under each tag.
- **Entries**: every entry in the period, oldest first.

The **Overview** shows the following:

- The hours worked in the period, with the change against the previous period of the same length and a line of the hours per day. Smaller figures sit next to it.
- Hours per day, week, or month, split by project.
- Project and tag shares. Each project also shows what its time earned, when it has a rate.
- The work you tracked most, grouped by Jira key.
- The average day per weekday, the time of day, and daily totals.
- One earnings total per currency, under **Hours by project** and in the daily totals. HourLog hides
  the line when no project in the period has a rate.

Turn on rounding to round each date, project, and task, and to see the difference against the
exact time. Rounding uses the step that you set in **Settings > Booking**. HourLog stores exact
times to the minute.

### See what your time earns

Hourly rates are off by default. To turn them on, go to **Settings > Modules** and turn on **Hourly
rates**. If you turn them off again, HourLog hides the rate fields and all earnings, and keeps the
rates that you entered.

Report exports leave money out by default, even with hourly rates on. To add rates, currencies, and
earnings to the summary CSV and the Excel workbook, turn on **Money in exports** under **Hourly
rates** in **Settings > Modules**. The full backup export always keeps your rates.

Set an hourly rate on a client, and optionally on a project, as described in
[Clients, projects, and tasks](#clients-projects-and-tasks). HourLog then works out the earnings of
your tracked time:

- A project uses its own rate if it has one. Otherwise it uses the rate of its client. A project
  rate of 0 counts as a rate and earns nothing.
- Earnings follow the hours that you book. With rounding on, they use the rounded hours of each
  date, project, and task. With rounding off, they use the exact time.
- Each amount is rounded to two decimals. A total is the sum of the rounded amounts, so the
  numbers on screen and in the exports always add up.
- A project without any rate shows a dash.
- The currency belongs to the client. A project is billed in the currency of its client, and a
  project without a client in euros. HourLog never adds amounts of different currencies. It shows one
  total for each currency instead.

HourLog calculates earnings from the current rates each time you open a report. It doesn't store
them per entry. If you change a rate, the earnings of past days change too, including days that you
booked.

### Export

The export menu on the **Analytics** page offers the following files:

- **Entries, one row each**: a CSV that you can import back. It has no earnings.
- **Summary by date, project and task**: a CSV with exact and rounded hours. With hourly rates and
  money in exports on, it adds the columns **Rate**, **Currency**, and **Earnings**. The three cells are empty for a row
  without a rate.
- **For booking, rounded with a memo**: the booking CSV described in
  [Export a week for booking](#export-a-week-for-booking).
- **Workbook with summary and entries**: an Excel file with a summary sheet and an entries sheet.
  Dates, durations, and hours are real Excel values, so you can sum and filter them. The summary
  sheet has an **Earnings** and a **Currency** column, and one total row per currency, when hourly
  rates and money in exports are on.
- **Summary report** and **Report with every entry**: PDF files. They show hours only, never money.

CSV files are UTF-8 with a byte order mark, so Excel shows diacritics correctly.

### Import from CSV

Click **Import CSV** on the **Analytics** page and pick a CSV file. HourLog reads the following:

- Any time-entry CSV with a start date and time, plus an end time or a duration. HourLog matches
  common column names such as Date, Start, End, Duration, Hours, Description, Project, Client,
  Task, and Tags.
- An HourLog entries CSV.

HourLog detects day-month or month-day dates, and 12-hour or 24-hour times. It creates missing
clients, projects, tasks, and tags. It skips rows that are already in HourLog, so a second run of
the same file adds nothing. One click undoes an import.

## Organize your work

### Clients, projects, and tasks

The **Projects** page lists clients and projects. You don't add tasks there. Tasks come in with
an import. A client can have a logo URL. HourLog shows the logo next to the client name everywhere.

To bill by the hour, turn on **Hourly rates** in **Settings > Modules**. Then enter an **Hourly
rate** and a **Currency** in the client dialog. The currency
is a three letter code such as EUR or USD, and EUR is the default. Leave the rate empty if you don't
bill the client by the hour. A project can have its own **Hourly rate** in the project dialog. It
replaces the client rate for that project and uses the currency of the client. Clear the field to go
back to the client rate. See [See what your time earns](#see-what-your-time-earns).

Some time isn't part of your working hours, such as on-call standby. To keep it out of the daily
and weekly targets, turn on **Don't count toward the daily and weekly targets** in the project
dialog. The project then shows an **Outside targets** badge. Its time doesn't fill the progress
bars in the header and the tray, and it doesn't count for the end-of-day reminder. It still counts
everywhere else: on the **Tracker**, **Timesheet**, and **Analytics** pages and in every export,
including the booking CSV.

Archive a project or task when you no longer use it. Archived items stay on old entries. Turn on
**Show archived** to see them.

To archive a project in one click, select the archive icon next to the pencil on its row. The toast
that follows has an **Undo** button. On an archived row, the same icon unarchives the project.

To delete a project, select the trash icon on its row. You can delete a project only when it has
no entries. Its planner items stay, without the project, and its routing rules go. The toast that
follows has an **Undo** button.

A project with entries can't be deleted, so its trash icon is dimmed. Select it to see why, then
merge or archive the project instead.

To merge two projects, select the merge icon on the row of the project that you want to remove.
Pick the project to merge into. The dialog shows how many entries move. HourLog moves the entries,
planner items, routing rules, and tasks, then deletes the first project. A task with the same name
in both projects becomes one task. You can't merge a project with entries on booked days. Unlock
those weeks on the **Timesheet** page first. The toast that follows has an **Undo** button.

### Tags

The **Tags** page manages labels such as `billable`, `meeting`, or `overtime`. Archived tags stay
on old entries.

If you don't use tags, go to **Settings > Modules** and turn off **Tags**. HourLog then hides the
Tags tab, the tag picker on the timer bar, entry rows, and the selection bar, and the tag filter,
tag grouping, tag column, and tag chart in Analytics. The tags on your entries stay. Turn **Tags**
on again to see them.

### Custom fields

The **Custom fields** page adds extra fields to entries, projects, or tasks. For example, add a
booking code to projects. Custom fields show up in reports and exports.

A field can be required. HourLog warns you about entries that miss a required field, and the
booking check lists them.

### Routing rules

The **Routing rules** page fills in the project and task from the description. A rule matches a
Jira key, a prefix, some text that the description contains, or a regular expression. For example,
a rule can send every description that starts with `Standup` to an internal project.

Jira key rules win. Then HourLog checks the other rules from the top of the list down. To see which
rule matches, type a description in the test field at the top of the page.

## Change settings

The **Settings** page has the following cards. On a wide window, a list of the cards stays on the
left while you scroll. Click a name to jump to that card. The list highlights the card that you're
looking at.

- **Appearance**: the theme, UI size, and accent color. Your browser stores this choice. The
  palette button in the header changes it too.
- **Language**: English or Romanian. **System** follows the language of your browser and of
  Windows. The choice applies to the app, error messages, reminders, the PDF and Excel reports,
  and the widget. CSV and JSON exports keep English column names, so other tools can read them.
- **Tracking**: the first day of the week, the time and duration formats, rounding when you stop a
  timer, daily and weekly targets, and whether every entry needs a project or a description.
- **Working hours and reminders**: your working days and hours, and these reminders:
  - No timer running, after a number of minutes. It repeats every hour.
  - A long-running timer, once per timer.
  - An end-of-day summary when today is under the daily target.
  - Desktop notifications from the browser. Leave them off when the tray icon runs, or you get
    each reminder twice.
- **Booking**: the rounding step for booked hours, and a weekly reminder to book the week. The
  reminder comes only when the week has time that isn't booked yet.
- **Backup**: HourLog backs up the database once a day while it runs. Click **Back up now** for an
  extra backup, or download a backup file.
- **Export and import everything**: one JSON file with all your data. Use it to move HourLog to
  another PC. Import adds only what is missing and makes a backup first.
- **Modules**: turn the planner, hourly rates, money in exports, and tags on or off.
- **Updates**: whether HourLog checks GitHub for new versions, the update status, **Check now**,
  and **Update**. See [Update from the app](#update-from-the-app).
- **About HourLog**: the version of the service, the account it runs as, its start time, and its
  time zone.

## Use the timer widget

`HourLogWidget.exe` is a small window that stays on top of other apps. It shows the running task,
its project, and the elapsed time, with a **Start** and **Stop** button. It talks to the service
directly, so it works with the browser closed.

- Drag the widget to move it. It remembers its position.
- Right-click the widget for these options: **Start** or **Stop**, **Open HourLog**,
  click-through, hide when no timer is running, opacity, and start with Windows.
- Press `Ctrl+Alt+H` to turn click-through on or off. With click-through on, your clicks reach the
  window below the widget. Opening the widget from the Start menu also turns click-through off.
- The widget follows the Windows light or dark app theme.
- The widget uses the language from **Settings**. It switches within a minute of a change.

The **Start** button starts a timer with no description. Add the details in the main app.

The widget also adds an HourLog icon to the notification area. The icon shows reminders, and its
menu can start or stop the timer, open HourLog, show the widget, or exit. The widget stores its
settings in `%LocalAppData%\HourLog\widget.json`.

## Configure the service

The service reads `appsettings.json` from its install folder. The `HourLog` section has these
keys:

| Key | Default | Description |
| --- | --- | --- |
| `Port` | `5280` | The port that serves the app and the API. |
| `DatabasePath` | Empty | The SQLite file. Empty means `%ProgramData%\HourLog\hourlog.db` on Windows, or `HourLog/hourlog.db` in the local app data folder elsewhere. |
| `TimeZoneId` | Empty | A time zone ID that the PC knows, such as `GTB Standard Time` on Windows. Empty means the time zone of the PC. |
| `UpdateRepository` | Set by the build | The GitHub repository, as `owner/name`, whose releases HourLog checks for updates. Empty turns update checks off, whatever **Settings** says. |

You can also set each key as an environment variable, such as `HourLog__Port`. Restart the
`HourLog` service after a change.

The service listens on `localhost` only, so other PCs can't open HourLog. It also rejects requests
from other websites in your browser. Only you and your local scripts can reach it.

## Troubleshoot

### The service doesn't start after an upgrade

The database migrations were reset to one `InitialCreate` migration on November 6, 2026. A database
made by an earlier build doesn't start, and the service stops with a message that names the file.

To fix this, move or delete that file, or point `HourLog:DatabasePath` to a new file. Then start
the service again. To keep your entries, export them as CSV from the old build first, and import
them in the new one.

### A change fails with "This day is booked"

The entry is on a booked day. Unlock the week on the **Timesheet** page, make the change, and book
the week again.
