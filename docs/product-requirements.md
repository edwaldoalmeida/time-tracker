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

V1 delivers a reliable log-entry and report workflow. The report is the primary outcome; logging provides the data it needs.

### Log entries

- Each entry represents a time block and includes:
  - `date` (mandatory)
  - `startTime` (optional)
  - `endTime` (optional)
  - `title` (optional)
  - `description + highlights` (optional)
  - `category` (mandatory)
  - `tags` (optional)
  - `feeling` (optional, not currently implemented but desired, to store how I felt during that activity)
- Examples of categories:
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
- Overlapping time entries are visibly flagged as an inconsistency, but the user can still save them.

### Main screen and suggestions

- Opening the app shows the log timeline.
- Older entries appear above newer entries; the entry composer is at the bottom.
- V1 suggests the current date and a start time rounded to 15-minute increments.
- V1 suggestions are limited to date and time defaults.

### Reports

- Time distribution reports by `category` and/or `tag` for a selected period
- Filtering log entries by `category` and/or `tag`
- Reporting periods include:
  - A selected calendar day
  - A calendar week (Monday through Sunday)
  - A calendar month
  - A calendar year
  - A custom start and end date
  - Since the last one-to-one
- The report is a chronological bullet list of highlights from log entries in the selected period.
- The report can be copied to the clipboard or saved as a Markdown (`.md`) file.

## Later capabilities

The following are explicitly outside V1: (but should be considered during data modeling / app design)

- Predictions for `titles`, `categories`, or `tags`
- Automatic positive/negative labeling of highlights
- MCP interface for communication with Hermes Agent
- Work location (WFH, office, or leisure)
- Learning notes per entry or end of day
