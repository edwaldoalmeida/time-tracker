# Software Design Specification

## STRATEGY
- Use the Hexagonal Architecture [https://netflixtechblog.com/ready-for-changes-with-hexagonal-architecture-b315ec967749]

## ARCHITECTURE AND BOUNDARIES

The application is intended for private access from multiple devices. The
frontend and backend are separately deployed services; PostgreSQL is the V1
database. A reverse proxy can expose the frontend and API under one address.

```text
Browser -> Frontend service -> Backend REST API -> Persistence port
                                                      |
                                              PostgreSQL adapter
                                                      |
                                                  PostgreSQL
```

The frontend calls a documented REST-style JSON API. The API defines the
endpoints, request and response formats, errors, and business behavior that a
replacement backend must preserve so the frontend works without changes.
A replacement frontend can use the same API and backend rules.

The backend is one Spring Boot application with clear internal boundaries:

- **Domain:** business concepts and rules, independent of Spring, HTTP, and
  storage technology.
- **Application:** use cases and ports that coordinate domain behavior and
  access external capabilities.
- **Adapters:** REST controllers and persistence implementations that
  translate between external technologies and the application.

Dependencies point inward from adapters to application to domain. The backend
is authoritative for validation, overlap detection, and report calculations.
The frontend may duplicate checks for immediate feedback, but the backend
validates requests independently.

For example, a REST adapter passes an entry request to an application use
case, which applies domain rules and accesses saved entries through a
persistence port. The PostgreSQL adapter implements that port. Invalid time
ranges are rejected; overlaps are flagged but do not prevent saving.

Persistence ports must not expose PostgreSQL or ORM details to application or
domain code. They should permit an alternative storage model, such as document
storage or files, to be implemented later. V1 does not require an alternative
implementation. An eventual MCP adapter could call the same application use
cases; MCP is outside V1.

Verify these boundaries with domain tests independent of HTTP, Spring, and the
database; persistence-adapter tests against port expectations; REST contract
tests that a replacement backend can also run; and a small number of
end-to-end entry and report workflow tests.

## DATA MODEL

The domain model is independent of the PostgreSQL representation. V1 uses
separate relational records for entries, highlights, categories, and tags, and
a join table for entry–tag associations. Other persistence adapters may use
different storage layouts while preserving the same behavior.

- **Account:** UUID, login identifier, password hash, role (admin or user),
  enabled flag, and creation timestamp. The first account created at setup is
  the admin. Accounts are never deleted, only disabled.
- **Entry:** UUID, owning account, required local work date and category,
  optional local start and end times at minute precision, title, description,
  and feeling, creation and last-updated timestamps, and soft-deletion
  information.
- **Highlight:** UUID, nonblank text, parent entry, and creation timestamp. An
  entry has zero or more highlights. Editing a highlight preserves its creation
  order; UUID breaks ties between identical creation timestamps.
- **Category:** UUID, owning account, and name. Each entry references exactly
  one category belonging to the same account. Every account is seeded with its
  own copy of the seven PRD categories, which are fixed for V1.
- **Tag:** UUID, owning account, and display name. Names match case-insensitively
  within an account. Tags are reusable and remain available even when no
  entries use them.
- **Entry–tag association:** Entries may have multiple tags, and tags may be
  used by multiple entries of the same account. Tags on an entry are unordered
  and cannot be duplicated.

All entries, categories, and tags belong to exactly one account, and every
read and write is scoped to the authenticated account. No account can read or
modify another account's data, including the admin.

Work dates and times preserve the entered local calendar and clock values;
access from another timezone does not shift them. Creation and update
timestamps are separate from these work values. Entries with the same start
time, and entries without a start time, are ordered by creation timestamp,
with UUID as a stable tie-breaker.

Surrounding whitespace is trimmed from text. Blank optional text is treated
as absent; a blank highlight input creates no highlight record and does not
prevent saving an entry. Feeling is optional and uses the V1 choices
`positive`, `neutral`, and `negative`.

Deleting an entry is a soft deletion: the entry and its highlights remain
stored but are excluded from normal timeline, report, and overlap queries.
V1 has no restore interface. Deleting an entry does not delete reusable tags.

The backend enforces the PRD's required fields and time rules as well as the
rules above. PostgreSQL constraints also protect relationships and
uniqueness, including entry–tag associations and case-insensitive tag names
within an account. Flyway manages the schema. Database backups must include
soft-deleted entries and retained tags.

Tests should cover persistence and retrieval of relationships, case-insensitive
tag matching, blank highlights, soft deletion, ordering ties, preservation of
local work dates and times, and rejection of access to another account's data.

## API CONTRACT

The REST-style JSON API uses the `/api/v1` prefix. Its OpenAPI specification
is written contract-first and defines the shared compatibility target for
frontends and replacement backends. The detailed contract is described in the
next section; the OpenAPI document itself is the first implementation
deliverable.

All API operations except login and the first-account setup require an
authenticated session. Results are scoped to the authenticated account, and
an operation on another account's resource behaves as if it does not exist.
Account management operations are restricted to admins.

The API provides operations to:

- List entries in paginated timeline order, loading the most recent batch
  first and allowing older batches to be retrieved.
- Get an entry, create one, replace its complete editable state with `PUT`,
  and soft-delete it.
- List fixed V1 categories and list or search reusable tags.
- Generate a report for a period type and selection, returning the resolved
  inclusive date range, all matching entries, totals, breakdowns, highlights,
  and an indication of entries without calculated duration.

Entry saves include the complete highlight list. Existing highlights retain
their IDs and creation order; omitted existing highlights are removed, and new
highlights receive IDs. The client supplies tag names; the backend matches
existing tags case-insensitively or creates missing ones. Saving the entry and
any new tags is atomic.

Entry reads and saves provide current overlap warnings. Editing or deleting
an entry can affect other entries' warnings, so the frontend refreshes affected
timeline data. Deleted entries remain excluded as described in the data model.

Individual entry reads return an `ETag` representing editable entry state,
including highlights and tags. Updates and deletes require `If-Match`; the
backend checks the token atomically with the change and rejects missing or
stale preconditions. Changes to an entry or its highlights change its token.

Reports follow the PRD's filtering, duration, overlap, and multi-tag rules.
Each report represents a consistent snapshot: its matching entries, totals,
and highlights agree even if data changes during generation. The frontend
formats the structured report for display and Markdown export.

Errors use a consistent JSON structure with a machine-readable code, a
human-readable message, and field-specific details where applicable. HTTP
status codes distinguish invalid requests, missing entries, and concurrency
conflicts. Contract tests cover endpoints, payloads, errors, and representative
business behavior; replacement backends should pass the same tests.

## TIME AND REPORTING RULES

The composer defaults the date to the device's current local date and leaves
the start time blank. Work times use minute precision. Durations are whole
minutes calculated from entered clock times, ignoring daylight-saving changes.
Entries cannot span midnight; when both times are present, the end must be
later than the start.

Overlap checks compare entries on the same work date only. Complete blocks
overlap when they intersect, not when they merely meet at a boundary. Against
a complete block, a start-only time is flagged at the block's start or inside
it, but not at its end; an end-only time is flagged inside it or at its end,
but not at its start. Two incomplete entries of the same type with the same
single time are flagged. Opposite types at the same time and incomplete
entries with different single times are not flagged against one another.

Reports include incomplete entries without adding them to duration totals,
and count entries without a calculated duration. Overlapping entries each
contribute their full duration. Each entry contributes once to the overall
total and its full duration to each applicable tag row. With selected tags,
only those tags appear in the breakdown; without a tag filter, the breakdown
includes all tags on matching entries and an Untagged group for matching
untagged entries. Groups without matching entries are omitted. A report with
no matches has zero overall duration, no groups or highlights, and a
no-matching-entries indication.

Custom report ranges are inclusive; reversed start and end dates are swapped.
Report highlights follow timeline entry order and then highlight creation
order, including the specified UUID tie-breakers. Other filtering and calendar
period rules follow the PRD.

## SECURITY AND OPERATING CONTEXT

The app runs on a machine on the owner's home network, reachable over the LAN
or a VPN. It is not exposed to the public internet.

HTTPS uses a self-signed or internal certificate. Each device that uses the
app trusts the certificate once. The hostname can be a subdomain of
`edwaldoalmeida.ca` resolved by local DNS.

Accounts are real user accounts with login. The first account created at
setup becomes the admin. There is no public registration; admins create
accounts. Admins can disable accounts but cannot delete them, so account data
is never orphaned. Admins manage accounts only and cannot read or edit other
users' entries.

Login sets an HTTP-only session cookie marked `Secure` and `SameSite`, so the
frontend never stores credentials or tokens. Passwords are stored using a
modern password hash such as Argon2id or bcrypt. Secrets and configuration are
supplied through environment variables and are not committed to the
repository.

A nightly `pg_dump` backs up the database to a separate disk or network
location. The restore procedure is documented and tested occasionally, and
includes soft-deleted entries and retained tags.

## TESTING STRATEGY

- **Backend unit tests (JUnit):** Domain rules with no Spring, HTTP, or
  database—overlaps, ordering, durations, report calculations, and validation.
- **Backend integration tests (JUnit + Testcontainers):** The persistence
  adapter against real PostgreSQL, including relationships, soft-deletion
  filtering, case-insensitive tag matching, per-account isolation, and schema
  constraints.
- **API contract tests:** Conformance to the OpenAPI document plus
  representative behavior, written so a replacement backend can run the same
  suite.
- **Frontend unit tests:** Components and state logic. The tool choice is still
  open.
- **System tests (Python/Pytest):** Multi-step workflows through the public API
  against a running stack—for example, entries across several days followed by
  a report whose totals are verified end to end.
- **End-to-end tests (Cucumber-Gherkin + Playwright):** Core workflows plus
  edge cases—login, entry create/edit/delete with tags and highlights, reports,
  overlap warnings, incomplete entries, stale-edit conflicts, and Markdown
  export. Each scenario runs as a unique fresh account, so scenarios share no
  data and need no database resets.
- **CI:** Fast checks on every push and PR (lint, unit, integration, contract).
  The E2E suite runs nightly or on demand.

## DETAILED API CONTRACT

Login is the only unauthenticated API operation. The first admin account is
created by a CLI setup step outside the API, not by an endpoint. All results
are scoped to the authenticated account; another account's resource returns
`404`.

### Endpoints

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/v1/session` | Log in; sets the session cookie |
| `DELETE` | `/api/v1/session` | Log out |
| `GET` | `/api/v1/session` | Current account, role, and forced-password-change flag |
| `POST` | `/api/v1/session/password` | Change own password; clears the forced-change flag |
| `GET` | `/api/v1/accounts` | List accounts (admin) |
| `POST` | `/api/v1/accounts` | Create account with a temporary password (admin) |
| `POST` | `/api/v1/accounts/{id}/disable` | Disable account (admin) |
| `POST` | `/api/v1/accounts/{id}/enable` | Re-enable account (admin) |
| `POST` | `/api/v1/accounts/{id}/password-reset` | Issue a new temporary password (admin) |
| `GET` | `/api/v1/entries` | Paginated timeline, newest batch first |
| `POST` | `/api/v1/entries` | Create entry; returns `201`, `Location`, and `ETag` |
| `GET` | `/api/v1/entries/{id}` | Get entry with `ETag` and inconsistencies |
| `PUT` | `/api/v1/entries/{id}` | Replace entry; requires `If-Match` |
| `DELETE` | `/api/v1/entries/{id}` | Soft-delete entry; requires `If-Match` |
| `GET` | `/api/v1/categories` | The account's categories |
| `GET` | `/api/v1/tags` | The account's tags, with optional `query` filter |
| `GET` | `/api/v1/reports` | Structured report for a period and optional filters |

Tags have no standalone creation endpoint. Saving an entry creates any missing
tags, matching existing names case-insensitively, in the same transaction.

Sessions last about 30 days of inactivity and end at logout. New accounts are
created with a temporary password and must change it before other operations
succeed.

### Representations

An entry response:

```json
{
  "id": "3f1c…",
  "date": "2026-11-24",
  "startTime": "09:00",
  "endTime": "10:30",
  "category": { "id": "8a2b…", "name": "Dev" },
  "title": "Fix login flow",
  "description": "Traced the cookie issue",
  "feeling": "positive",
  "tags": [{ "id": "c7d1…", "name": "ASTRA" }],
  "highlights": [{ "id": "b4e9…", "text": "Found the root cause" }],
  "inconsistencies": [{ "code": "overlap", "relatedEntryIds": ["9f0c…"] }],
  "createdAt": "2026-11-24T14:02:11Z",
  "updatedAt": "2026-11-24T14:05:43Z"
}
```

Dates are `YYYY-MM-DD`, times are `HH:mm`, and timestamps are ISO-8601 UTC.
Absent optional text or times are `null`. On write, `categoryId` identifies the
category and `tags` is a list of names. Highlights carry an `id` when they
already exist and omit it when new. `inconsistencies` is read-only and names
the entries involved so the UI can show what conflicts.

### Pagination

The timeline response is `{ "entries": [...], "nextCursor": "…" }`. A request
without a cursor returns the newest batch; `nextCursor` returns the batch
immediately older and is `null` when no older entries remain. Each batch is
ordered oldest to newest internally, so the frontend can render it as-is and
prepend it above earlier batches.

The cursor is opaque and encodes the position of the oldest item in the batch
using the timeline ordering key: date, start time, creation timestamp, and UUID.

### Concurrency

The `ETag` is a strong validator over the entry's editable state, including
highlights and tags; any change to either changes the token. `PUT` and `DELETE`
require `If-Match`, and the token is checked atomically with the change.
Missing or stale preconditions return `412`.

### Reports

```
GET /api/v1/reports?period=week&date=2026-11-24
GET /api/v1/reports?period=custom&startDate=2026-11-01&endDate=2026-11-15
GET /api/v1/reports?period=month&date=2026-11-01&categoryId=8a2b…&tagIds=c7d1…&tagIds=e3f4…
```

`period` is `day`, `week`, `month`, `year`, or `custom`. For calendar periods,
`date` selects the containing period. `categoryId` and repeated `tagIds` are
optional filters. The response contains the resolved period, the applied
filters, `overallDurationMinutes`, `entriesWithoutDuration`, a
`categoryBreakdown`, a `tagBreakdown`, `highlights`, and the matching
`entries`. Breakdowns follow the inclusion rules in the time and reporting
section; an empty report has zero durations and empty arrays, and the frontend
shows the no-matching-entries message.

### Errors and status codes

Errors use `application/problem+json` per RFC 9457, with field-level detail as
an extension:

```json
{
  "type": "https://time-tracker/problems/validation",
  "title": "Request validation failed",
  "status": 422,
  "detail": "The entry has invalid values.",
  "errors": [
    { "field": "endTime", "code": "must-be-after-startTime", "message": "End time must be later than start time." }
  ]
}
```

| Code | Meaning |
|---|---|
| `200` | Read, update, or report succeeded |
| `201` | Entry or account created, with `Location` |
| `204` | Delete or logout succeeded |
| `400` | Malformed request (bad JSON, unknown parameters) |
| `401` | Not authenticated, or session expired |
| `403` | Authenticated but not allowed (non-admin, or password change required) |
| `404` | Not found, including another account's resources |
| `409` | Duplicate account identifier |
| `412` | Missing or stale `If-Match` |
| `422` | Request understood but failed business validation |

## PERFORMANCE TESTING

Performance testing answers three questions: how the app behaves with years of
data, how it behaves under concurrent sessions, and how its usual techniques
work in practice. JMeter runs the load tests; k6 is noted as a possible later
experiment.

Test data has two sizes:

- **Baseline:** about five years of normal use, roughly 6,000–7,000 entries
  with tags and highlights.
- **Stress ceiling:** about ten years of heavy use, roughly 35,000–40,000
  entries.

Three workload profiles run against each dataset:

- **Read-heavy:** timeline paging and report generation.
- **Mixed realistic:** open the timeline, page through several batches, create
  a few entries, and open a report.
- **Write-heavy:** repeated entry saves, stressing tag resolution and
  concurrency tokens.

Concurrency ramps from one session to 100 or more, recording where response
times degrade.

Pass/fail thresholds apply at baseline volume only, at p95. The stress ceiling
is exploratory and only reports results.

| Operation | Target (p95, baseline) |
|---|---|
| Timeline page (50 entries) | < 300 ms |
| Get a single entry | < 150 ms |
| Create an entry with tags and highlights | < 400 ms |
| Report, month period | < 1.5 s |
| Report, year period | < 3 s |

## TECHNOLOGY AND DEPLOYMENT DECISIONS

- **Frontend:** React with TypeScript, built by Vite. Unit tests use Vitest.
  No Redux: state stays in components and a small fetch layer, since most state
  is server data.
- **Backend:** Java with Spring Boot. Persistence uses Spring Data JPA
  (Hibernate) behind the persistence ports. Schema migrations use Flyway, with
  Liquibase noted as a possible later experiment.
- **Database:** PostgreSQL.
- **Deployment:** Docker Compose on one home machine, running the frontend,
  backend, PostgreSQL, and a TLS reverse proxy. Migrating to K3s is a planned
  later experiment; the images and configuration carry over.
- **Versions:** Use current LTS releases for the JDK, Node, Spring Boot, and
  PostgreSQL, pinned explicitly in build files and container images.

## TECH STACK

### frontend
  - React
  - TypeScript
  - Vite
  - no Redux: component state plus a small fetch layer
  * unit tests
    - Vitest

### backend
  - Java
  - Spring/Spring Boot
  - Spring Data JPA (Hibernate)
  - Flyway (migrations)
  * tests
    - JUnit (unit and integration)
    - Testcontainers (PostgreSQL)
    - JMeter (performance testing; k6 as a possible later experiment)

### database
  - PostgreSQL

### system tests
  - Python + Pytest

### UI/end-to-end tests
  - Cucumber-Gherkin driving Playwright

### deployment
  - container-based
  - Docker Compose for V1
  - K3s migration planned as a later experiment

### general
  - data layer: ORM to be able to swap database (and to learn how to work with ORMs)
  - architecture: approach such as that I'd be able to easily swap backend or frontend
