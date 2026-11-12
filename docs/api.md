# Local API reference

HourLog serves a REST API at `http://localhost:5280/api`. The web UI, the timer widget, and your
own scripts all use it. This page lists every endpoint.

## Conventions

- The service listens on the loopback address only. Other PCs can't reach it.
- The API has no sign-in.
- The service refuses browser calls from other websites. A request with an `Origin` header that is
  not `localhost` gets `403`. Scripts such as `curl` and PowerShell send no `Origin` header, so they
  pass.
- Bodies and responses are JSON in camelCase. Enums are strings, for example `"Add"`.
- Times are ISO 8601 with an offset, for example `2026-11-06T09:30:00+02:00`. Dates are
  `yyyy-MM-dd` local dates.
- Durations in responses are whole seconds. Entries start and end on whole minutes, so their
  durations are multiples of 60. HourLog drops the seconds of a start or an end that you send, and
  rounds a duration to the nearest minute.
- Validation errors return `400` with a problem detail and an `errors` map of field names to
  messages.
- A missing item returns `404`.
- A change to an entry on a booked day returns `409`. See [Booking](#timesheet-and-booking).

The C# records behind these shapes are in `src/HourLog.Application/Contracts`. The TypeScript
copies are in `src/HourLog.Web/src/lib/types.ts`.

## Example

Start a timer from PowerShell:

```powershell
Invoke-RestMethod -Method Post http://localhost:5280/api/timer/start `
  -ContentType 'application/json' -Body '{"description":"JIRA-7596 Release .NET 10"}'
```

Read last week's booking rows as CSV:

```bash
curl "http://localhost:5280/api/booking/export?from=2026-11-02&to=2026-11-08"
```

## Timer

| Method | Path | Body | Returns |
| --- | --- | --- | --- |
| GET | `/timer` | | The running `EntryDto`, or `204` when no timer runs |
| POST | `/timer/start` | `{ description, projectId?, taskId?, tagIds? }` | `SaveResult`. Stops any running timer first |
| POST | `/timer/stop` | | `SaveResult`, or `204` when no timer runs |
| POST | `/entries/{id}/continue` | | `SaveResult` with a new running entry that copies the details |
| GET | `/timer/sleep` | | `SleepGapDto` when the PC slept while the timer ran, or `204` |
| POST | `/timer/sleep` | `{ action }` | `204`. See the actions below |

The sleep actions are:

- `Keep`: count the time the PC slept.
- `Discard`: stop the timer at the moment the PC went to sleep.
- `DiscardAndContinue`: stop the timer at that moment and start it again when the PC woke up.

## Entries

| Method | Path | Body | Returns |
| --- | --- | --- | --- |
| GET | `/entries?from&to&q` | | `EntryDto[]` that start in `[from, to)` |
| GET | `/entries/week?date` | | `WeekDto`: days, grouped entries, and totals |
| GET | `/entries/{id}` | | `EntryDto` |
| POST | `/entries` | `EntryInput` | `SaveResult` |
| PUT | `/entries/{id}` | `EntryInput` | `SaveResult` |
| DELETE | `/entries/{id}` | | `{ changeBatchId }` |
| POST | `/entries/{id}/duplicate` | | `SaveResult` |
| POST | `/entries/{id}/split` | `{ at }` | Both halves |
| POST | `/entries/{id}/round-up` | `{ minutes? }` | `SaveResult`. Moves the end so the duration rounds up to the next step |
| POST | `/entries/bulk` | `{ ids, projectId?, taskId?, tagIds?, delete? }` | `{ changeBatchId, count }` |
| POST | `/undo/{changeBatchId}` | | `204` |
| GET | `/suggestions?q&limit` | | Earlier descriptions with their project, task, and tags |

`EntryInput` is `{ description, date?, start, end?, duration?, projectId?, taskId?, tagIds?,
customFields? }`:

- `start` and `end` take `HH:mm` together with `date`, or a full ISO time.
- If `end` is earlier than `start`, the entry ends the next day.
- `duration` takes `1.5`, `1:30`, `90m`, `1h30`, or `1h 30m`. A bare number means hours.
- `customFields` maps a custom field id to its value. Leave it out on an update to keep the stored
  values.

`SaveResult` is `{ entry, overlaps, changeBatchId }`. `overlaps` lists other entries that share
time with the saved one. Overlaps are allowed. Pass `changeBatchId` to `/undo` to revert the
whole change.

## Projects, clients, tags, and settings

| Method | Path | Body |
| --- | --- | --- |
| GET | `/projects?includeArchived` | |
| POST | `/projects` | `{ name, clientId?, color?, customFields?, hourlyRate?, excludedFromTargets? }`. With `excludedFromTargets` on, the time of the project doesn't count in `/progress` |
| PUT | `/projects/{id}` | The POST body plus `archived`. A missing `hourlyRate` removes the project rate |
| PUT | `/projects/{id}/archived` | `{ archived }`. Changes only the archived flag and skips the custom field checks |
| DELETE | `/projects/{id}` | Deletes a project with no entries. Returns `{ changeBatchId, entries }`. A project with entries gives 400 |
| POST | `/projects/{id}/merge` | `{ targetId }`. Moves the entries, planner items, rules, and tasks to the target, then deletes the project. Returns `{ changeBatchId, entries }`. Entries on booked days give 400 |
| POST | `/projects/{id}/tasks` | `{ name, customFields? }` |
| PUT | `/tasks/{id}` | `{ name, archived, customFields? }` |
| GET | `/clients` | |
| POST | `/clients` | `{ name, logoUrl?, hourlyRate?, currency? }` |
| PUT | `/clients/{id}` | `{ name, archived, logoUrl?, hourlyRate?, currency? }`. A missing `logoUrl` removes the logo, a missing `hourlyRate` removes the rate, and a missing `currency` means `EUR` |
| GET | `/tags?includeArchived` | |
| POST | `/tags` | `{ name }` |
| PUT | `/tags/{id}` | `{ name, archived }` |
| GET | `/settings` | |
| PUT | `/settings` | The full `SettingsDto` |

`tagsEnabled` in `SettingsDto` is `true` by default. When it is `false`, the UI hides tags. The tag
endpoints, the `tagIds` fields, and the exports still work, and stored tags stay on their entries.

`moneyInExports` in `SettingsDto` is `false` by default. It lets the `summary-csv` and `xlsx` report
exports carry rates and earnings when `hourlyRatesEnabled` is on too. The full export ignores it.

`language` in `SettingsDto` is `system`, `en`, or `ro`. It sets the language of error messages
(`title`, `detail`, and the field errors), reminders, and the PDF and Excel reports. `system` follows
the Windows display language. CSV and JSON exports stay English.

### Hourly rates

A client has an optional `hourlyRate` and a `currency`. A project has an optional `hourlyRate` that
overrides the client rate. The currency belongs to the client only, so a project is always billed in
the currency of its client. A project without a client uses `EUR`. The `ClientDto` and `ProjectDto`
return these fields.

- `hourlyRate` is `null` (no rate) or a number from 0 to 100000 with at most two decimals. A rate of
  0 is a rate: it wins over the client rate and earns nothing.
- `currency` is a three letter code such as `EUR` or `USD`. HourLog stores it in upper case and checks
  the shape only, not a list of codes. An empty or missing value means `EUR`.
- A bad value returns `400` with the field name `hourlyRate` or `currency`.

## Custom fields

| Method | Path | Body |
| --- | --- | --- |
| GET | `/custom-fields?includeArchived` | |
| POST | `/custom-fields` | `{ name, type, level, options?, required }` |
| PUT | `/custom-fields/{id}` | The POST body plus `archived`. You can't change `type` or `level` |

- `type` is `Text`, `Number`, `List`, or `Checkbox`.
- `level` is `Entry`, `Project`, or `Task`.
- Validation errors use keys such as `customFields.Location`.

## Routing rules

Routing rules fill in the project and task of an entry from its description.

| Method | Path | Body | Returns |
| --- | --- | --- | --- |
| GET | `/routing-rules` | | Rules in the order they apply |
| POST | `/routing-rules` | `{ matchType, pattern, projectId, taskId?, enabled }` | The new rule |
| PUT | `/routing-rules/{id}` | The POST body | The rule |
| DELETE | `/routing-rules/{id}` | | `204` |
| POST | `/routing-rules/reorder` | `{ ids }` with every id once | The rules in the new order |
| GET | `/routing/resolve?description` | | `{ ruleId, projectId, taskId }`, or `204` when no rule matches |

`matchType` is `JiraKey`, `Prefix`, `Contains`, or `Regex`. The first enabled rule that matches
wins.

## Reports, export, and import

Every report takes these query parameters:

- `from` and `to`: local dates, both included, at most five years apart.
- `clients`, `projects`, `tasks`, and `tags`: optional comma-separated ids. In `clients`, the empty
  GUID means "no client." In `projects`, it means "no project."
- `booked`: optional. `true` keeps entries on booked days only. `false` keeps entries on days that
  aren't booked.
- `q`: optional text or Jira key.
- `rounding`: optional step in minutes, from 0 to 60. `0` turns rounding off.

| Method | Path | Extra query | Returns |
| --- | --- | --- | --- |
| GET | `/reports/analytics` | | `AnalyticsDto`: key figures, days, projects, tags, weekdays, hours, top activities, and one row per date, project, and task |
| GET | `/reports/detailed` | | `DetailedReportDto`: the entries that start in the period |
| GET | `/reports/export` | `format`, `filterText` | A file. `format` is `csv`, `summary-csv`, `xlsx`, `pdf`, or `pdf-entries` |
| POST | `/import` | `fileName` with a raw body | `ImportResultDto`: counts, issues per row, and a `changeBatchId` for undo |
| GET | `/import/history` | | The last 50 imports |

Notes:

- Reports split entries that cross midnight and count only the days in the period.
- Rounding applies to each date, project, and task row.
- `hourlyRatesEnabled` echoes the setting of the same name, which is `false` by default. When it is
  `false`, every rate and earnings value is `null`, `earningsTotals` is empty, and the `summary-csv`
  and `xlsx` exports leave out the rate and earnings columns. The client and project endpoints still
  accept and return rates, and the full export keeps them.
- `moneyInExports` is `true` only when both `hourlyRatesEnabled` and the `moneyInExports` setting
  are on. The setting is `false` by default. When `moneyInExports` is `false`, the `summary-csv` and
  `xlsx` exports leave out the rate, currency, and earnings columns and the per-currency totals, even
  though the analytics response still carries earnings. The PDF reports never show money.
- Each row, and each project in `projects`, has `earnings` and `currency`, plus `hourlyRate` on a row.
  They are `null` when the project and its client have no rate. Earnings are the rate times the
  rounded hours of the row (the exact hours when rounding is off), rounded to two decimals, halves away
  from zero. A project's earnings are the sum of its rounded rows.
- `earningsTotals` has one `{ currency, amount }` per currency, sorted by currency. HourLog never adds
  amounts of different currencies. Rows without a rate count in no total.
- Earnings are computed from the current rates each time. They are not stored per entry, so changing a
  rate also changes the earnings of past and booked days.
- With `moneyInExports` on, the `summary-csv` file adds the columns `Rate`, `Currency`, and `Earnings` after `Difference (h)`
  and before the custom field columns. Rows without a rate leave the three cells empty. A tool that
  reads the file by column position after the first nine columns must account for the shift. The
  `csv` (entries) and booking files are unchanged.
- The `xlsx` workbook has two sheets. **Summary** has one row per date, project, and task, with
  totals. **Entries** has one row per entry. Dates, times, durations, and hours are Excel values,
  not text. The Summary sheet ends with the columns **Earnings** (a number, empty without a rate) and
  **Currency**. The Total row leaves them empty. Under it, one **Total** row per currency sums the
  earnings with `SUMIF`.
- `POST /import` takes a CSV file as the multipart field `file` or as the raw body, up to 20 MB.
- The import skips a row when the same row was imported before, or when an entry with the same
  start, end, and description exists. It also skips rows on booked days.

## Timesheet and booking

| Method | Path | Body or query | Returns |
| --- | --- | --- | --- |
| GET | `/timesheet?date` | Any day of the week. Defaults to today | `TimesheetDto` |
| PUT | `/timesheet/cell` | `{ date, projectId, taskId, seconds, mode }` | `{ timesheet, changeBatchId }` |
| GET | `/booking` | | `{ periods, openWeeks }` |
| POST | `/booking` | `{ from, to }`, at most 31 days | `201` with the booked period |
| DELETE | `/booking/{id}` | | `204` |
| GET | `/booking/check?from&to` | | `{ from, to, issues }` |
| GET | `/booking/export?from&to&rounding&skipIncomplete` | | The booking CSV |

### Timesheet cells

`TimesheetDto` has a row per project and task. Each row has seven cells with `seconds`,
`entryCount`, and `running`. `entryCount` counts the entries that start on that day.

When you set a cell, the service does the following:

- If the cell is empty, it adds one manual entry. The entry copies the description, tags, and entry
  fields from the latest entry of the row. It starts where the tracked time of that day ends, or at
  the start of the working day.
- If the cell has entries, `mode` decides:
  - `Add` adds the difference as a new entry. The new value must be higher.
  - `AdjustLast` moves the end of the latest entry of the cell.
  - `Replace` deletes the entries of the cell and adds one entry with the new value. `0` deletes
    them.

The whole edit is one change batch, so `/undo/{changeBatchId}` reverts it.

### Booked periods

- `POST /booking` fails with `400` when the days overlap a booked period, or when the running timer
  started on one of them.
- While a period is booked, any write that adds, changes, or deletes an entry on one of its days
  returns `409`. Delete the period to unlock it.
- `openWeeks` lists the finished weeks of the last eight that have tracked time and are not booked.
- A full import, which restores a backup, can bring back entries on booked days.

### Booking check and export

`/booking/check` lists entries that have no project, or that miss a required entry, project, or
task field.

`/booking/export` returns one CSV row per date, project, and task with these columns:

- `Date`, `Client`, `Project`, and `Task`.
- One column per project field and per task field, such as `Project field: Booking code`.
- `Hours`: rounded to `rounding` minutes. The default is the booking rounding setting, 15 minutes.
- `Exact hours`.
- `Memo`: the activities of the row with their share of the rounded hours, for example
  `LOCK-12 Locker firmware (1.25), Code review (0.75)`. An activity is the first Jira key of an
  entry, or its description. The shares add up to the rounded hours of the row.

Rows that round to zero are left out. With `skipIncomplete=true`, the export leaves out the entries
that the check lists.

## Planner

| Method | Path | Body | Returns |
| --- | --- | --- | --- |
| GET | `/plan?from&to` | | Days in the range (at most 62) with items and totals, the backlog, and the overdue count |
| POST | `/plan` | `PlanItemInput` | The new item |
| PUT | `/plan/{id}` | `PlanItemInput` | The item |
| POST | `/plan/{id}/move` | `{ date, position? }`. A null `date` moves the item to the backlog | The item |
| POST | `/plan/{id}/copy` | `{ date, position? }`. Without `position`, a copy on the same day goes right below the original | `201` and the new item. The copy starts not done |
| DELETE | `/plan/{id}` | | `204`. A soft delete |
| POST | `/plan/{id}/restore` | | The restored item |
| POST | `/plan/calendar/preview?from&to` | A multipart `.ics` file | The meetings in the range. Saves nothing |
| POST | `/plan/import` | `{ items }` | `{ created }`. Saves all items or none |
| POST | `/plan/roll-over` | | `{ moved, to }`. Moves unfinished items of past days to today |

The planner API stays on when you turn the planner off in **Settings**.

## System, progress, backups, and full export

| Method | Path | Returns |
| --- | --- | --- |
| GET | `/system` | Version, commit, runtime, operating system, Windows service flag, start time, and time zone |
| GET | `/progress` | Tracked seconds today and this week, the targets, and whether a timer runs. Projects with `excludedFromTargets` don't count, and a timer on one gives `isRunning: false` |
| GET | `/backups` | Database path and size, backup folder, retention, last and next backup, and the backups |
| POST | `/backups` | `201` with the new backup file |
| GET | `/backups/{name}` | The backup file. Only names like `hourlog-20261106-093000.db` are served |
| GET | `/export/full` | Everything as one JSON file |
| POST | `/import/full` | `{ counts, backupName }` |

The full export has `format: "hourlog-full-export"` and `version: 2`. It holds the settings,
custom fields, clients, projects, tasks, tags, routing rules, entries, plan items, and booked
periods. It leaves out deleted entries and the undo history.

Version 2 added `hourlyRate` and `currency` to each client and `hourlyRate` to each project. The
import still reads version 1 files, where the missing fields mean no rate and `EUR`. It refuses a
file with a newer version, and an older HourLog refuses a version 2 file, so no rates are dropped
silently. It also refuses a file with a rate or currency that the catalog would not accept.

The full import takes the file as a multipart field `file` or as the raw body. It takes a backup
first, then adds only the rows that are missing. It matches rows by id, and clients, projects,
tasks, tags, and custom fields also by name. It never changes or deletes existing rows, so the rates
of a client or project that already exists stay as they are.

## Updates

| Method | Path | Returns |
| --- | --- | --- |
| GET | `/update` | The update status |
| POST | `/update/check` | Checks GitHub now and returns the status |
| POST | `/update/install` | `202` with the status. Backs up the database and starts the installer |

The status has these fields:

- `state`: `Idle`, `Checking`, `UpToDate`, `Downloading`, `Ready`, `Installing`, or `Failed`.
- `currentVersion` and `latestVersion`. `latestVersion` is set only when a newer release exists.
- `releaseNotes`, `releaseUrl`, and `publishedAt` of that release.
- `downloadBytes` and `downloadedBytes` while the installer downloads.
- `lastCheckedAt` and `error`.
- `checkEnabled`: whether the background check runs.
- `canInstall`: `true` when the installer is ready and the service runs as a Windows service.

`/update/install` answers `400` when no update is ready, when the downloaded file no longer matches
its checksum, or when the service doesn't run as a Windows service.

## Live updates

`GET /events` is a server-sent events stream with three events:

- `ready`: sent when you connect.
- `changed`: sent after every write. Bursts of writes collapse into one event.
- `notify`: a reminder as JSON, `{ kind, title, message, at }`.

`kind` is one of `no-timer`, `long-timer`, `end-of-day`, `sleep`, or `week-not-booked`. The service
checks reminders every 30 seconds and follows the working days, hours, and thresholds in the
settings.
