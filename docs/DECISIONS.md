# DECISIONS — architecture decision log

> Short, append-only record of the decisions that shape the build, so collaborators see the *why* without re-reading every doc. Newest first. Each entry: decision · why · consequence.

## ADR-009 — AI-ready by construction (2026-06)
**Decision:** Customization documents (layouts/themes/rules/config) are the AI's API; an AI assistant is just another client of the validated propose→preview→approve pipeline. On-device Apple Foundation Models are the default; external AI is opt-in with consent; core actions are exposed as App Intents. See [`AI-READINESS.md`](AI-READINESS.md).
**Why:** The data-not-code model already makes the content AI-legible; baking in schemas + a capability manifest + App Intents now avoids a costly retrofit later.
**Consequence:** Two cheap obligations pulled into Phases 1–3 (schema-first edit API; wrap actions as App Intents). AI output is re-validated like any document; external AI requires Guideline 5.1.2(i) in-app consent; nothing executes as code (2.5.2).

## ADR-008 — Performance is a gate, not a phase (2026-06)
**Decision:** Every PR meets the budgets in [`PERFORMANCE.md`](PERFORMANCE.md); customization compiles once into typed Swift and the hot path is plain SwiftUI.
**Why:** Deep customization is the main jank/bloat risk; "slim, fast, smooth" only holds if it's enforced continuously.
**Consequence:** Perf tests in CI; budgets in Definition of Done; a slow feature gets redesigned, not shipped.

## ADR-007 — De-risk with spikes before committing (2026-06)
**Decision:** Prototype the four risky unknowns (SDUI perf, multi-CKShare tenancy, rules dedup, customization fallback) before full implementation. See [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) §2.
**Why:** The engines are the only novel parts; discovering a blocker late kills velocity.
**Consequence:** Phase 0/1 includes throwaway spikes, each answering one yes/no question.

## ADR-006 — Public release brings UGC duties (2026-06)
**Decision:** A public, multi-tenant app implements App Store Guideline 1.2/1.2.1 (EULA, report, block, moderation, age-gating); group admins are first-line moderators.
**Why:** Required for any app hosting user content; was skippable only while TestFlight-only.
**Consequence:** A dedicated Phase 4 moderation workstream; RBAC must grant moderation powers.

## ADR-005 — Customization is data, not code (2026-06)
**Decision:** Layouts and rules are declarative documents interpreted by engines compiled into the binary; never downloaded/executed code, never an HTML5/JS mini-app.
**Why:** App Store Guideline 2.5.2 forbids downloaded executable code; 4.7 (mini-apps) adds a regime we don't want.
**Consequence:** Fixed component registry + fixed action allow-list as safety boundaries; SDUI renders native SwiftUI.

## ADR-004 — Roles are app-level RBAC over CKShare (2026-06)
**Decision:** `owner`/`admin`/`member`/`invited` stored per group and enforced in the data layer; CKShare provides only the access boundary.
**Why:** CKShare models participant read/write, not admin/moderator semantics, which the pivot needs.
**Consequence:** A roles record per group; every privileged action re-checked server-side, not just hidden in UI.

## ADR-003 — One CKShare zone per group (multi-tenant) (2026-06)
**Decision:** Each friend group is its own CloudKit CKShare zone; a user participates in many. WAD?FC is seeded via the normal create/invite flow, never hardcoded.
**Why:** The pivot serves any friend group; "membership = the one zone" no longer works.
**Consequence:** Cost rides each user's iCloud quota (keeps it ad-free at scale); the zone boundary is the security wall.

## ADR-002 — Full SDUI + rules engine (the pivot) (2026-06)
**Decision:** Users customize layouts and behavior, not just colors/fonts. Two engines: [`SDUI-SPEC.md`](SDUI-SPEC.md) and [`RULES-SPEC.md`](RULES-SPEC.md). Core jobs ship as the default template.
**Why:** "Every group has special needs and aesthetics" — the product is a customizable platform.
**Consequence:** Biggest architectural change; screens/behavior become versioned data with graceful fallback.

## ADR-001 — iOS-only for now, portable later (2026-06)
**Decision:** Stay iOS-only; keep customization *documents* platform-neutral so Android via Skip needs only a second renderer, not a format change.
**Why:** Deep Apple integration + speed now; preserve the project's Android goal without paying for it yet.
**Consequence:** CloudKit-centric backend stays; cross-platform is a future renderer/evaluator, or Firebase fallback.
