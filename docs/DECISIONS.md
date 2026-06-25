# DECISIONS — architecture decision log

> Short, append-only record of the decisions that shape the build, so collaborators see the *why* without re-reading every doc. Newest first. Each entry: decision · why · consequence. ADR-010+ are the **idea-phase direction** ([`VISION.md`](VISION.md)) — proposed, to be validated by spikes before code. **Strategic forks live in [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md).**

## ADR-015 — Strategic positioning is OPEN, pending Ruben (2026-06)
**Decision:** Don't lock niche-vs-mass, Android scope, or monetization yet — frame them as explicit open decisions ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) O1–O3) for Ruben/the crew. Recommendations recorded (us-first; Android "thin seam"; monetization deferred) but **not locked**.
**Why:** An external critique correctly flagged strategic ambiguity as the top risk; the cheapest fix is to surface the forks, not bury them. We're in planning, no code yet.
**Consequence:** `PIVOT.md` (multi-tenant) and `TECH-STACK.md` (heavy portability) are **proposed**, not locked, on these axes until decided.

## ADR-014 — One model = semantically unified, physically separated (2026-06)
**Decision:** Conceptually one object graph, but `View`/`Reaction`/`Theme` are **independent bounded modules behind interfaces** — not one mega-renderer. Unification lives in envelope/storage/sync/validation only. See [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L2.
**Why:** Avoids the Spotify-HubFramework coupling/performance trap a critique flagged, while keeping the conceptual elegance.
**Consequence:** [`DATA-MODEL.md`](DATA-MODEL.md)/[`ARCHITECTURE.md`](ARCHITECTURE.md) treat the three as separate implementations over one model.

## ADR-013 — Settings/editor-first; AI is a later connector (2026-06)
**Decision:** v1 customization is a **settings/creator surface** (options + token editing + paste scripts/config), **not** conversation-first AI. The only AI work now is **futureproofing** (validated declarative documents = schema-as-API). See [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L1.
**Why:** Conversation-first UX + on-device-AI capability are unvalidated; don't bet v1 on them. The data-not-code design already makes AI a later "just another client."
**Consequence:** [`VISION.md`](VISION.md) Inversion 3 reframed; [`AI-READINESS.md`](AI-READINESS.md) is a futureproofing spec, not a v1 feature; [`CREATOR.md`](CREATOR.md) Script depth is the v1 power surface.

## ADR-012 — Motion & designs are declarative, portable bundles (2026-06, proposed)
**Decision:** Motion/animation is customizable as **data** (motion tokens + Lottie/Rive-style declarative files), never code. A whole look (theme + tokens + motion + assets + layout + rules) exports/imports as an open **bundle** via the iOS Files app / links. A public store, if wanted, can be **third-party** — we don't run one. See [`CREATOR.md`](CREATOR.md) §11, [`FEASIBILITY.md`](FEASIBILITY.md).
**Why:** Full expressive customization (even motion, even "bring your own") while staying App-Store-safe (2.5.2) and portable; sharing without us hosting/moderating a store.
**Consequence:** An open bundle format + schema validation + review-before-run for any rules inside; assets validated and slim; Reduce-Motion + per-frame budgets enforced.

## ADR-011 — One domain-typed object model (2026-06, proposed)
**Decision:** Unify the plumbing around a single reactive object graph where **views** (layout), **reactions** (rules), and **looks** (theme/motion) are facets of the same domain-typed objects (`Event`, `Poll`, `Tradition`…) — not three separate engines, and **not** generic primitives. See [`VISION.md`](VISION.md) Inversion 2. (Refined by ADR-014: physically separated modules.)
**Why:** Fewer concepts to build/teach, ideal substrate for AI, composability. Domain typing avoids the Spotify-HubFramework over-abstraction trap ([`PRIOR-ART.md`](PRIOR-ART.md)).
**Consequence:** [`SDUI-SPEC.md`](SDUI-SPEC.md) becomes the *view* facet, [`RULES-SPEC.md`](RULES-SPEC.md) the *reaction* facet, over one store.

## ADR-010 — Local-first, CloudKit as transport (2026-06, proposed)
**Decision:** The group's state is a local-first, event-sourced/CRDT store on each device; **CloudKit is the sync transport + record store, not the source of truth**; R2 holds blobs. Instant + offline by default; sync merges conflict-free. See [`VISION.md`](VISION.md) Inversion 1, [`REALTIME.md`](REALTIME.md). **Plan B if CRDT proves too hard:** CloudKit as source of truth + optimistic updates ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3).
**Why:** Delivers slim/fast/smooth structurally (no spinners, offline), resilience, ownership, and breaks the CloudKit lock-in (sync is a swappable port).
**Consequence:** Biggest new engineering ([`CRITIQUE.md`](CRITIQUE.md) §1.2–1.3); use a proven CRDT library; bound the log with snapshots/compaction; near-real-time chat free, sub-second presence is the one paid-ish edge.

## ADR-009 — AI-ready by construction (2026-06)
**Decision:** Customization documents (layouts/themes/rules/config) are the AI's API; an AI assistant is just another client of the validated propose→preview→approve pipeline. On-device Apple Foundation Models are the default; external AI is opt-in with consent; core actions are exposed as App Intents. See [`AI-READINESS.md`](AI-READINESS.md). (Per ADR-013, AI itself is a *later* feature; this is futureproofing.)
**Why:** The data-not-code model already makes the content AI-legible; baking in schemas + a capability manifest + App Intents now avoids a costly retrofit later.
**Consequence:** Two cheap obligations pulled into early phases (schema-first edit API; wrap actions as App Intents). AI output is re-validated like any document; external AI requires Guideline 5.1.2(i) in-app consent; nothing executes as code (2.5.2).

## ADR-008 — Performance is a gate, not a phase (2026-06)
**Decision:** Every PR meets the budgets in [`PERFORMANCE.md`](PERFORMANCE.md); customization compiles once into typed Swift and the hot path is plain SwiftUI.
**Why:** Deep customization is the main jank/bloat risk; "slim, fast, smooth" only holds if it's enforced continuously.
**Consequence:** Perf tests in CI; budgets in Definition of Done; a slow feature gets redesigned, not shipped.

## ADR-007 — De-risk with spikes before committing (2026-06)
**Decision:** Prototype the risky unknowns (SDUI perf, multi-device tenancy, rules dedup, customization fallback, local-first merge) before full implementation. See [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) §2 and [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3.
**Why:** The engines are the only novel parts; discovering a blocker late kills velocity.
**Consequence:** Spikes run first, each answering one yes/no question; nothing built until they pass.

## ADR-006 — Public release brings UGC duties (2026-06)
**Decision:** *If* we go multi-tenant public (O1), the app implements App Store Guideline 1.2/1.2.1 (EULA, report, block, moderation, age-gating); group admins are first-line moderators.
**Why:** Required for any app hosting user content; skippable while us-first/TestFlight.
**Consequence:** A dedicated moderation workstream gated on O1; RBAC must grant moderation powers.

## ADR-005 — Customization is data, not code (2026-06)
**Decision:** Layouts and rules are declarative documents interpreted by engines compiled into the binary; never downloaded/executed code, never an HTML5/JS mini-app.
**Why:** App Store Guideline 2.5.2 forbids downloaded executable code; 4.7 (mini-apps) adds a regime we don't want.
**Consequence:** Fixed component registry + fixed action allow-list as safety boundaries; SDUI renders native SwiftUI.

## ADR-004 — Roles are app-level RBAC over the access boundary (2026-06)
**Decision:** `owner`/`admin`/`member`/`invited` enforced in the data layer, not just UI. (Per O1: inside a group, keep permissions light/social — [`VISION.md`](VISION.md) Inversion 4; full RBAC only matters if multi-tenant.)
**Why:** The access boundary models in/out, not admin/moderator semantics.
**Consequence:** A roles record per group; every privileged action re-checked.

## ADR-003 — One zone per group (multi-tenant capable) (2026-06, proposed)
**Decision:** Each group is its own CloudKit CKShare zone; a user participates in many. WAD?FC is seeded via the normal create/invite flow, never hardcoded. (Scope gated on O1.)
**Why:** Serves any friend group if we go that way; "membership = the one zone" no longer works.
**Consequence:** Cost rides each user's iCloud quota; the zone boundary is the security wall.

## ADR-002 — Full SDUI + rules engine (the pivot) (2026-06)
**Decision:** Users customize layouts and behavior, not just colors/fonts. Two engines: [`SDUI-SPEC.md`](SDUI-SPEC.md) and [`RULES-SPEC.md`](RULES-SPEC.md). Core jobs ship as the default template. (Reframed by ADR-011/014 as separated facets of one model.)
**Why:** "Every group has special needs and aesthetics" — the product is a customizable platform.
**Consequence:** Biggest architectural change; screens/behavior become versioned data with graceful fallback.

## ADR-001 — iOS-only for now, portability TBD (2026-06)
**Decision:** Stay iOS-only; how much portability to build is open (O2, leaning "thin seam"). Keep customization *documents* platform-neutral regardless.
**Why:** Deep Apple integration + speed now; preserve the Android option cheaply.
**Consequence:** CloudKit-centric backend stays; cross-platform is a future renderer/adapter if O2 says so.
