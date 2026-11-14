# Product Requirements

## Product goals

- Replace the spreadsheet used throughout the workday with a quicker way to record and review time.
- Help prepare for one-to-one meetings with a concise summary of recorded highlights.

## Engineering goals

- Use the project as a lab for practicing software engineering and experimenting with technology choices, while keeping the app useful for daily tracking.

## V1 scope

V1 delivers a reliable log-entry workflow and a report for preparing for one-to-ones. The report is the primary outcome; logging provides the data it needs.

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
- Start and end times are not required for every entry.
- Categories are mandatory. Examples:
  - `QA`
  - `Dev`
  - `Misc`
  - `Meeting`
  - `DayOff`
  - `Commute`
- Tags are optional, that can be created on the fly/while entering data. Examples:
  - `VC-NNNN`
  - `VC9.4`
  - `9.4.0b2`
  - `local-llms`
  - `ASTRA`
- A one-to-one is an ordinary log entry whose tag is a consistent `1:1` or `one-to-one`. The title may also include the other person's name.
- Each highlight has text and a positive or negative label (`feeling`).
- Users can edit or delete saved log entries and their highlights.
- Overlapping time entries are visibly flagged as an inconsistency, but the user can still save them.

### Main screen and suggestions

- Opening the app shows the log timeline.
- Older entries appear above newer entries; the entry composer is at the bottom.
- V1 suggests the current date and a start time rounded to 15-minute increments.
- When the start time is more than 15 minutes old, V1 suggests the current time rounded to 15-minute increments as the end time.
- V1 suggestions are limited to date and time defaults. Title, category, and type/ticket predictions or history-based suggestions are deferred.

### Reports

- Time distribution by category/tag for a selected period (week, month, year, or other range)
- Time-distribution reports by category or type for a selected period
- Filtering log entries by category or type

## Later capabilities

The following are explicitly outside V1: (but should be considered during data modeling / app design)

- AI predictions for titles, categories, or types/tickets
- Automatic positive/negative labeling of highlights
- MCP interface for communication with Hermes Agent
- Work location (WFH, office, or leisure)
- Learning notes per entry or end of day

## Future report ideas

- One-to-one highlights report
  - The report is a chronological bullet list of highlights from log entries in the selected period.
  - If the selected period contains no highlights, the report shows an empty-state message rather than blocking generation.
  - It includes both positive and negative highlights and uses only recorded highlights; it must not invent achievements, events, or details.
  - V1 may automatically rephrase highlights into concise bullets. The user can edit the generated bullets before copying or saving them.
  - The rephrasing provider is configurable, so the user can choose a local or external AI service.
  - If automatic rephrasing is unavailable or fails, the report remains usable with the original highlight wording.
  - The report can be copied to the clipboard or saved as a Markdown (`.md`) file.
  - Reporting periods include:
    - A selected calendar day
    - A calendar week (Monday through Sunday)
    - A calendar month
    - A calendar year
    - A custom start and end date
    - Since the last one-to-one
  - “Since the last one-to-one” includes entries after the most recent earlier entry with the `1:1` title marker through today, excluding the one-to-one entry itself. If no one-to-one entry exists, it includes all logged entries so far.
