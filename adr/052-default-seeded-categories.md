# ADR-052: Default seeded categories (from the gap-fill catalog) + category emoji

**Status:** Proposed
**Date:** 2026-07-01
**Deciders:** Mert Ertugrul

## Context

[ADR-048](048-categories-client-model-and-live-synced-management.md) established
categories as server-owned domain data that **start empty**: a new user sees no
categories until they create one by hand. User testing of the Categories and
gap-fill features (Jul 2026) showed this cold-start is a dead end — the slice
dialog hides its category section entirely when the list is empty (FE
[#371](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/371)), so a new
user never discovers tagging and has nothing to tag with.

Meanwhile [ADR-045](045-gap-fill-activity-suggestions.md) already curates a set of
activities — derived from the in-code `TemplateLibrary`
([ADR-051](051-app-curated-in-code-day-templates.md)) — with sensible labels,
colours, and a productivity/break classification. That same set is the natural
seed for a user's starting categories: it is opinionated, already colour-balanced,
and consistent with what the gap-fill card suggests.

Two decisions follow from "ship users a starting set of categories":

- **Where the seed happens.** Categories are shared, server-owned entities with an
  identity and a name-uniqueness rule (ADR-048). A client-side seed would race
  across devices and reinstalls (two devices both seeding, or a reinstall
  re-seeding after a deliberate clear).
- **How categories are told apart at a glance.** The gap-fill catalog carries a
  per-label emoji (🧠 Deep Work, 🎨 Creative, …) but the `Category` model
  (ADR-048) has no emoji field, so seeded categories would render as colour-only.

## Decision

Seed a fixed set of default categories **on the backend at account creation**,
backfill existing empty accounts once, and **add an optional `emoji` field to the
`Category` model** so the seed carries its catalog emoji. Every default is an
ordinary category — fully renameable, recolourable, and deletable.

- **Seed set = the full gap-fill catalog.** The 15 entries below (13 distinct
  `TemplateLibrary` labels, first-occurrence-wins, plus the `Focus`/`Recharge`
  fallbacks) become the default categories. Name → colour → emoji:

  | Name | Colour | Emoji |
  |------|--------|-------|
  | Deep Work | `#43AA8B` | 🧠 |
  | Lunch | `#F9C74F` | 🍽️ |
  | Creative | `#AD5D9B` | 🎨 |
  | Wind-down | `#577590` | 🌙 |
  | Gym | `#F94144` | 💪 |
  | Work | `#3E5C76` | 💼 |
  | Dinner | `#F9C74F` | 🍝 |
  | Unwind | `#48BFE3` | 🛋️ |
  | Study | `#43AA8B` | 📚 |
  | Break | `#F8961E` | ☕ |
  | Review | `#6A994E` | 📝 |
  | Reading | `#577590` | 📖 |
  | Prep for Sleep | `#48BFE3` | 😴 |
  | Focus | `#43AA8B` | 🎯 |
  | Recharge | `#48BFE3` | ☕ |

  Repeated colours are intentional and expected (Deep Work/Study/Focus share teal);
  the user recolours freely afterward.

- **Backend seeds at account creation.** The server inserts the set when a user
  record is created, inside the same transaction, so the categories exist before
  the client's first fetch. Cross-device consistency and the name-uniqueness
  constraint are enforced server-side, and there is no client seeding code — the FE
  keeps its "render whatever the server returns" stance (ADR-048).

- **Backfill existing empty accounts once, guarded.** A one-off, idempotent backfill
  seeds the same set for any existing user whose category list is empty **and who
  has never been seeded** — distinguished by a per-user `categoriesSeeded` flag set
  the first time the seed runs. A user who created categories and then deleted them
  all has the flag set and is left alone; only genuinely never-seeded users are
  backfilled. Re-running the backfill is a no-op.

- **`Category` gains an optional `emoji`.** The model/API add a nullable `emoji`
  string (ADR-048's colour stays as `#RRGGBB`). Seeded categories carry their
  catalog emoji; user-created categories may set one via an emoji picker in Manage
  Categories, or leave it null. The field degrades to "no emoji" on absent/malformed
  data, matching the tolerant-parse posture of ADR-048. Category chips and the slice
  dialog render `emoji + name` when an emoji is present, name-only otherwise.

- **Seeded categories are not special.** They emit the normal
  `CATEGORY_CREATED` STOMP events (ADR-048), participate in optimistic
  edit/rollback, and carry no "is default" marker — once seeded they are
  indistinguishable from hand-created categories, so edits and deletes behave
  identically.

## Alternatives Considered

- **Client-side seed on first empty fetch (guarded by a local flag).** Ships without
  a backend change, but the guard is per-device: a second device or a reinstall
  after a deliberate clear re-seeds, and two devices racing both write the set. A
  server seed is authoritative and single-sourced.
- **No seed; rely on the empty-state prompt (FE #371) alone.** Keeps ADR-048's
  empty-start model, but leaves every new user to build categories from scratch —
  the exact friction testing flagged. The prompt stays as the fallback for a user
  who clears their list, but is not a substitute for a starting set.
- **Seed a smaller curated subset.** Less overwhelming, but arbitrary — the gap-fill
  catalog is already the app's curated set, and shipping the same list keeps
  categories, suggestions, and templates telling one story. Users delete what they
  don't want.
- **Backfill all empty users unconditionally (no `categoriesSeeded` flag).** Simpler,
  but repopulates a user who intentionally emptied their list on their next sync —
  a surprising, unwanted write. The flag is the minimum state needed to respect a
  deliberate clear.
- **Skip the emoji field; seed colour-only.** Smaller change, but the seeded set
  loses the visual identity the gap-fill catalog already defines, and categories
  stay harder to scan. A nullable field is cheap and backward-compatible.

## Consequences

- New accounts start with 15 usable, colour- and emoji-distinct categories; the
  slice dialog's category section is populated from first launch, and gap-fill
  suggestions line up with the user's actual categories out of the box.
- ADR-048's "categories start empty" invariant is **superseded**: the FE must no
  longer treat an empty list as the normal new-user state (though it must still
  handle empty gracefully — a user can delete every category). FE #371's empty-state
  prompt becomes the rarer post-clear fallback rather than the default first-run
  experience.
- Backfill is a one-time, idempotent migration gated on a new per-user
  `categoriesSeeded` flag; the "never had vs deleted-all" distinction lives entirely
  in that flag.
- `Category` carries a new optional `emoji`; the cache JSON shape and API payload
  gain a nullable field that degrades to null on old/malformed data, so old clients
  and pre-migration rows keep working.
- The default set is a soft contract shared between the backend seed and the FE
  `TemplateLibrary`/`SuggestionCatalog` it was derived from. They can drift; a later
  cleanup could make the FE catalog derive from the seeded categories, but that is
  out of scope here.

*Related: BE [#335](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/335),
FE [#371](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/371),
[ADR-045](045-gap-fill-activity-suggestions.md),
[ADR-048](048-categories-client-model-and-live-synced-management.md) (superseded in
part), [ADR-051](051-app-curated-in-code-day-templates.md)*
