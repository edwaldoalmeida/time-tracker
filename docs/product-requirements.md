# Product Requirements

## Problem and context

Work time is currently tracked in an Apple Numbers spreadsheet. Adding entries,
creating weekly headers, and moving items around to keep the spreadsheet
organized all require manual effort. The app should make it easier to record
daily work activities and understand how time has been spent.

## Product goals

- Replace the spreadsheet used throughout the workday with a quicker way to record and review time.
- Help prepare for one-to-one meetings with a concise summary of recorded highlights.

## Engineering goals

- Use the project as a lab for practicing software engineering and experimenting with technology choices, while keeping the app useful for daily tracking.

## Constraints

- Development time is approximately one hour per day.
- The frontend, backend, and database should be replaceable so different technologies can be explored and compared in the same project.

## V1 scope

V1 delivers a reliable log-entry and report workflow.

### Log entries

- Each entry represents a time block and includes:
  - `date` (mandatory)
  - `startTime` (optional)
  - `endTime` (optional)
  - `title` (optional)
  - `description` (optional)
  - `highlights` (optional)
  - `category` (mandatory)
  - `tags` (optional)
  - `feeling` (optional, to store how I felt during that activity)
- The fixed V1 categories are:
  - `QA`
  - `Dev`
  - `Misc`
  - `Meeting`
  - `DayOff`
  - `Commute`
  - `SickDay`
- Tags can be created on the fly/while entering data. Examples:
  - `VC-NNNN`
  - `VC9.4`
  - `9.4.0b2`
  - `local-llms`
  - `ASTRA`
- Users can add, edit or delete saved log entries and their highlights.
- Saved entries persist across app restarts.
- Entries cannot span midnight. When both times are provided, `endTime` must be later than `startTime`.
- Overlapping time entries are visibly flagged as an inconsistency, but the user can still save them. Entries that meet at a time boundary do not overlap.
- An entry with only a start or end time is flagged when that time falls within another entry's time block.

### Main screen and suggestions

- Opening the app shows the log timeline.
- All entries appear on the timeline, with older dates above newer dates; the entry composer is at the bottom.
- Within a date, entries with a start time are ordered by start time. Entries without a start time follow them in creation order.
- For each new entry, the app suggests:
  - the current date for each new entry,
  - a start time based on the previous entry's end time, rounded down to a 15-minute increment. If there is no entry for the current day, or the previous entry has no end time, use the current time rounded down instead.

### Reports

- One time-distribution report shows results by `category` and/or `tag` for a selected period, along with highlights from matching entries.
- Filter log entries by `category` and/or `tag`. Multiple selected tags match any selected tag. When both a category and tags are selected, entries must match the category and at least one selected tag.
- Entries without both a start and end time remain in the results but do not contribute to duration totals; the report indicates that some entries lack a calculated duration.
- Overlapping entries each contribute their full duration, even when this counts overlapping time more than once.
- An entry can have multiple tags. Its full duration counts under each of its tags in a tag breakdown, but only once in the overall total. The report explains that tag totals may exceed the overall total.
- Entries without highlights remain in the results and contribute to time totals when their duration can be calculated.
- Reporting periods include:
  - A selected calendar day
  - A calendar week (Monday through Sunday)
  - A calendar month
  - A calendar year
  - A custom start and end date, both inclusive
- Use the device's local timezone for the current date and report boundaries.
- The report includes a chronological bullet list of highlights from matching log entries. Each highlight includes the entry's date, category, and title when present.
- The report can be copied to the clipboard or saved as a Markdown (`.md`) file, including the time distribution and highlights.

## Later capabilities

The following are explicitly outside V1: (but should be considered during data modeling / app design)

- Predictions for `titles`, `categories`, or `tags`
- Automatic positive/negative labeling of highlights
- MCP interface for communication with Hermes Agent
- Work location (WFH, office, or leisure)
- Learning notes per entry or end of day
