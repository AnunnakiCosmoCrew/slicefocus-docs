# ADR-025: Use Architecture Decision Records for technical decisions

**Status:** Accepted
**Date:** 2026-03-18
**Deciders:** Mert Ertugrul

## Context

As the project grows, significant technical decisions are made in PRs, Linear comments, and Slack threads — but these are hard to find later. New contributors (human or AI) lack context on *why* a pattern was chosen, leading to repeated discussions or accidental reversals. The project needed a lightweight, version-controlled way to capture and reference architectural decisions.

## Decision

Adopt Architecture Decision Records (ADRs) as the standard practice for documenting significant technical decisions. ADRs live in the [slicefocus-docs](https://github.com/AnunnakiCosmoCrew/slicefocus-docs) repository under `adr/`, numbered sequentially (e.g., `025-short-title.md`). Each ADR follows a consistent template: Status, Date, Deciders, Context, Decision, Alternatives Considered, and Consequences.

**When to write an ADR:**
- Choosing a framework, library, or infrastructure component
- Changing an established architectural pattern
- Making a trade-off that future developers will question
- Any decision where "why didn't we just…?" is likely to come up

**When NOT to write an ADR:**
- Routine implementation details or bug fixes
- Decisions that are easily reversed (e.g., variable naming)

ADRs are never deleted. If a decision is reversed, the original ADR is updated to "Superseded by ADR-NNN" and a new ADR captures the new direction.

## Alternatives Considered

- **Wiki pages**: Rejected because wikis lack PR review and version history. Easy to go stale.
- **Inline code comments**: Rejected because they scatter rationale across files and don't capture cross-cutting decisions.
- **No formal process**: Rejected because the project already lost context on several decisions (e.g., timer architecture, rendering approach) that had to be re-discovered.

## Consequences

- Decisions are discoverable via a single index (`adr/README.md`)
- AI agents (Claude Code, Copilot) can reference ADRs for context before proposing changes
- Adds a small overhead per decision (~10 min to write), offset by time saved re-explaining choices
- Historical decisions (ADR-001 through ADR-024) retroactively documented from Linear issues and PR history
