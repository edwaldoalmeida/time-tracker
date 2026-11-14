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

- **Entry:** UUID, required local work date and category, optional local start
  and end times at minute precision, title, description, and feeling, creation and last-updated
  timestamps, and soft-deletion information.
- **Highlight:** UUID, nonblank text, parent entry, and creation timestamp. An
  entry has zero or more highlights. Editing a highlight preserves its creation
  order; UUID breaks ties between identical creation timestamps.
- **Category:** UUID and name. Each entry references exactly one category.
  The seven categories in the PRD are fixed for V1.
- **Tag:** UUID and display name. Names match case-insensitively. Tags are
  reusable and remain available even when no entries use them.
- **Entry–tag association:** Entries may have multiple tags, and tags may be
  used by multiple entries. Tags on an entry are unordered and cannot be
  duplicated.

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
uniqueness, including entry–tag associations. Schema migrations manage the
initial tables and future changes; the migration tool remains to be chosen.
Database backups must include soft-deleted entries and retained tags. The
backup schedule and restore procedure will be decided with deployment.

Tests should cover persistence and retrieval of relationships, case-insensitive
tag matching, blank highlights, soft deletion, ordering ties, and preservation
of local work dates and times.

## API CONTRACT

The REST-style JSON API uses the `/api/v1` prefix. Its OpenAPI specification
is written contract-first and defines the shared compatibility target for
frontends and replacement backends. Exact paths, schemas, status codes,
pagination mechanics, and examples will be specified in that contract.

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

## REMAINING DESIGN DECISIONS

The following are placeholders for future discussion; they are not yet V1
design decisions.

### Security and operating context

- Define the hosting environment, private access method, and network exposure.
- Decide authentication and authorization, TLS, secrets management, and data
  protection appropriate for access from multiple devices.
- Define configuration, persistent storage, backup schedule, and a tested
  restore procedure, including recovery of soft-deleted entries.

### Testing strategy

- Map requirements and failure cases to unit, persistence integration, API
  contract, system, and end-to-end tests.
- Decide the roles and tools for frontend, backend, system, end-to-end, and
  any performance testing; determine whether Python system tests are needed.
- Define how tests run locally and in continuous integration, including test
  data and database setup.

### Detailed API contract

- Write the contract-first OpenAPI specification: exact paths, schemas,
  status codes, error responses, pagination, concurrency, and examples.
- Specify compatibility and contract-test expectations for replacement
  backends.

### Technology and deployment choices

- Select frontend testing tools (Jest or Vitest) and end-to-end tools
  (Playwright, Cucumber, or both).
- Choose the ORM and schema migration tool; assess whether Redux is needed.
- Decide the initial container deployment approach and whether Kubernetes or
  K3s is a V1 requirement or a later experiment.
- Specify runtime and tool versions, configuration, and deployment workflow.

## TECH STACK

### frontend
  - React
  - Redux
  - TypeScript
  * unit tests
    - Jest (unit tests) / Vite + Vitest

### backend
  - Java
  - Spring/Spring Boot
  * unit tests
    - JUnit
    - JMeter ?

### database
  - PostgreSQL

### system tests
  - Python + Pytest

### UI/end-to-end tests
  - Cucumber-Gherkin / Playwright

### deployment
  - container-based
  - deployment in K8s (or K3s)

### general
  - data layer: ORM to be able to swap database (and to learn how to work with ORMs)
  - architecture: approach such as that I'd be able to easily swap backend or frontend
