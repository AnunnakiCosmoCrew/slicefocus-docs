# ADR-029: SharedPreferences over SQLite for Local Persistence

**Status:** Accepted
**Date:** 2026-03-18
**Deciders:** Mert Ertugrul

## Context

The app needs local persistence for several categories of data: user settings (theme, timer durations, sound preferences, auto-advance flag), the offline action queue (ADR-027), and sync dirty flags. All of this data is key-value in nature — there are no relational queries, no joins, and no complex indexing requirements. Introducing a database layer would add migration overhead and dependency complexity for what is fundamentally flat, small-volume data.

## Decision

Use `SharedPreferences` as the sole local persistence layer. No SQLite, Hive, Drift, or Isar.

Key design choices:
- **JSON serialization for complex objects:** Structured types like `TimerPreferences` are serialized to JSON strings and stored under a single key. This keeps the API simple while supporting nested data.
- **Self-healing on corrupt data:** If JSON deserialization fails (e.g., after an app update changes the schema), the corrupt key is removed and the default value is returned. This prevents crashes from stale or malformed local state.
- **Per-user namespacing:** Keys that are user-specific (e.g., offline queue, preferences) are prefixed with the user ID to support account switching.

## Alternatives Considered

- **SQLite / Drift:** Full relational database with schema migrations. Overkill for key-value settings — adds build-time code generation, migration files, and a heavier dependency footprint.
- **Hive:** Fast key-value store with binary serialization. Adds TypeAdapter boilerplate and a proprietary binary format that complicates debugging and data inspection.
- **Isar:** Powerful NoSQL database with indexing and queries. Heavyweight for simple preferences — designed for apps with thousands of records and complex query patterns.

## Consequences

- Simple and lightweight — SharedPreferences is a Flutter first-party plugin with zero setup
- No schema migrations needed; adding a new preference is just a new key
- Limited query capability — no filtering or sorting across records. Acceptable because all data access patterns are single-key lookups.
- Self-healing prevents crashes from corrupt state, at the cost of silently resetting user preferences in rare edge cases
- JSON serialization format for complex objects becomes a soft contract; breaking changes should include migration logic

*Related: MER-227, MER-231*
