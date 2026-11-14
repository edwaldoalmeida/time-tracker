## Project:
- (N)YATTA - (Not) Yet Another Time Tracking Application

## Problem: [Who has what pain, and why does it matter?]
- I use to keep track of how I spend my time at work
- I use an Apple Numbers spreadsheet, where I manually:
  - add new entries
  - create headers for each week
  - move things around to keep it organized

## Goal: [What does "done" look like? One sentence.]
- An app to:
  - register my daily activities at work
  - get reports and insights about how I've spent my time

## In Scope (v1)
~~* Data entry: `date` | `startTime` | `endTime` | `title` | `category` | `type/ticket` | `description` | `highlights`~~
* Data entry: `date` | `startTime` | `endTime` | `title` | `description + highlights` | `category` | `tags` | `feeling`
* Reports per period: (dateA -> dateB)
  * time allocation per `category` (quick filter)
  * time allocation per `tag` (quick filter) (can also be done when filtering first by `category`, then by `tag`)
  * summary of time spent / achievements per period

## Out of Scope (for now) [Common overreach area 1] [Nice-to-have, not needed yet]
* smart auto-complete
* AI insights
* MCP / agent integration

## Constraints
* Time: 1 hour/day project
* Tech: ability to swap frontend/backend/database easily, so I can play with different tech in the same project/environment/sandbox and compare them

## Notes / Open Questions [Anything uncertain]
* ...
