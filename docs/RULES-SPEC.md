# RULES-SPEC — the rules / automation engine ("advanced settings")

> **Status: design spec (proposed, June 2026).** Defines the programmable "advanced settings" layer. Companion to [`PIVOT.md`](PIVOT.md) and [`SDUI-SPEC.md`](SDUI-SPEC.md). Implementation lands in roadmap Phase 3.
>
> **Performance:** evaluation must be invisible per [`PERFORMANCE.md`](PERFORMANCE.md) §4 — off the main thread, indexed by trigger (no linear scans), coalesced/debounced, bounded, and deduplicated so work happens on one device not nine.

## 1. Goal and the non-negotiable constraint

Let groups automate their own app — "**when** something happens, **if** some condition holds, **do** these actions" — from a friendly builder for everyone and a raw editor for power users.

Same hard line as the layout engine: **the evaluator is compiled into the binary; only declarative rule *data* is stored and synced.** Rules are content, not code (App Store **2.5.2**). The DSL is intentionally **not** a general programming language — it has no loops, no recursion, no arbitrary I/O, and a fixed action allow-list, so it can't become an escape hatch for downloaded behavior.

## 2. Shape of a rule

Trigger → (optional) conditions → ordered actions, expressed as data:

```jsonc
{
  "schema": 1,
  "id": "weekend-venue-poll",
  "enabled": true,
  "trigger": { "on": "event.created" },
  "when": {                                   // all must hold (AND); arrays of these allowed
    "all": [
      { "field": "event.dayOfWeek", "op": "in", "value": ["Sat", "Sun"] },
      { "field": "event.hasVenue",  "op": "eq", "value": false }
    ]
  },
  "do": [
    { "action": "createPoll",
      "with": { "question": "Where?", "options": "{event.suggestedVenues}" } },
    { "action": "notify",
      "with": { "audience": "going", "text": "Vote on the venue for {event.title}" } }
  ],
  "limits": { "maxRunsPerDay": 5 }            // per-rule safety cap
}
```

## 3. Triggers

Two families:

- **Event-driven** (fire on a domain data change): `event.created`, `event.updated`, `rsvp.changed`, `poll.created`, `poll.closed`, `message.posted` (rate-limited), `member.joined`, `member.left`, `availability.changed`, `points.changed`.
- **Schedule-driven** (fire on time): `schedule.at` (a specific datetime), `schedule.cron` (e.g. every Monday 09:00 Europe/Oslo), `date.reached` (relative to an event, e.g. "1 day before"). All times resolve in **one group timezone** (default Europe/Oslo) to avoid per-device drift.

## 4. Conditions

A small boolean expression tree over a **documented field catalog** (the same data sources the SDUI bindings expose, plus the trigger payload). Operators: `eq`, `neq`, `in`, `notIn`, `gt`, `gte`, `lt`, `lte`, `contains`, `exists`. Combinators: `all` (AND), `any` (OR), `not`. No user-defined functions, no expression evaluation beyond this grammar — bounded by design.

## 5. Actions (the fixed allow-list)

An action can only be one of a **closed, reviewed set**, each with declared inputs and a declared effect:

`notify` (push/in-app to an audience role/segment) · `postMessage` · `createPoll` · `createEvent` · `awardPoints` · `setGroupSetting` (whitelisted settings only) · `addToBucketList` · `pinMessage` · `scheduleReminder`. New actions are added deliberately, each reviewed for abuse potential. There is **no** "run arbitrary thing," no network fetch, no file access, no code eval.

## 6. The evaluator: where it runs and how it stays safe

- **Mostly on-device.** When a client observes a triggering data change (via CloudKit sync), it evaluates matching rules. Scheduled rules use the existing **free serverless cron** to wake evaluation when no client is active.
- **Deterministic & bounded:** no loops/recursion in the DSL; per-rule `maxRunsPerDay` and a global per-group rate limit; a hard cap on actions per run; conditions short-circuit. A rule cannot consume unbounded resources.
- **Side-effect sandbox:** actions only touch the **current group's** data through the permission layer — a rule can never reach another group's zone or escalate the author's role.
- **Permission-gated authoring:** only roles allowed by the RBAC layer (owner/admin, optionally trusted members) can create/enable rules; every action a rule performs is re-checked against the *rule author's* effective permissions at run time.

## 7. The duplicate-execution problem (and the fix)

Because up to ~all members' devices see the same data change, naive evaluation would run an action many times (nine notifications, nine polls). The engine enforces **exactly-once-ish execution** via a **claim record**: before performing a rule's actions for a given trigger instance, a client writes a uniquely-keyed "claim" (rule id + trigger event id) to the group zone; whoever wins the write executes, others see the claim and skip. CloudKit's last-write-wins + a deterministic claim key makes this safe enough for our scale; actions are also designed to be **idempotent** where possible (e.g. "create poll for event X" no-ops if one exists).

## 8. Versioning, validation, fallback

- **Schema version** per rule; evaluator supports a range.
- **Validate on save and before run:** trigger/field/action exist in the catalog, types match, author has permission, limits present. Invalid → rule is disabled with an admin-visible error, never a crash.
- **Unknown trigger/action (older app, newer rule):** that rule is skipped safely by the older client, not partially executed.
- **Kill switch:** any admin can disable a single rule or all automation for the group instantly; a misbehaving rule is one toggle from off.

## 9. Storage and sync

Rule definitions, their enabled state, and an execution/claim log live in the **group's CloudKit zone** (the customization store), syncing to all members and respecting the zone boundary. The log gives admins an audit trail ("this rule fired, did these actions") and powers debugging.

## 10. The authoring surface (friendly + advanced)

- **Friendly builder:** a guided "When … If … Then …" UI with pickers for triggers, conditions, and actions — no JSON. Templates for common automations ("remind everyone the day before an event," "auto-poll the venue").
- **Advanced editor:** raw rule-document editing with live validation, a dry-run/simulate mode against recent data, and the execution log — the power-user "scripts" surface. Gated by RBAC.
- **Shareable rules:** export/import rules as part of the Phase 4 template gallery; an imported rule is validated and shown for review **before** it's allowed to run (abuse-scanning is an open decision in [`PIVOT.md`](PIVOT.md) §10).

## 11. Relationship to the layout engine

Rules and layouts are complementary halves of "make this app yours": SDUI changes **what the app looks like**, rules change **what the app does**. They share the field/data-source catalog, the permission layer, the per-group customization store, and the same "data-not-code, validate-and-degrade" safety philosophy.
