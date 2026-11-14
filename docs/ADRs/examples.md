# Example ADRs for your tech stack:

## ADR-001: Backend Language Selection

Context

We are building a personal project tool that needs to be developed quickly, is easy to maintain, and leverages my existing skills.

Decision

We will use Python 3.12 as the primary backend language.

Consequences

• ✅ Fast development cycle, large ecosystem of libraries
• ✅ I'm already proficient in Python (primary skill)
• ❌ Slower runtime performance compared to Go or Rust
• ❌ Global Interpreter Lock (GIL) limits true multi-threading

## ADR-002: Database Selection

Context

The project requires persistent data storage. It's a personal tool with moderate read/write operations and no concurrent multi-user access expected.

Decision

We will use SQLite as the primary database.

Consequences

• ✅ Zero configuration, file-based, easy backup (just copy the .db file)
• ✅ Python has built-in sqlite3 — no external dependencies
• ❌ No concurrent write support (acceptable for single-user)
• ❌ Not horizontally scalable (fine for personal use)
