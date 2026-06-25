# DECISIONS — architecture decision log

> Short, append-only record of the decisions that shape the build, so collaborators see the *why* without re-reading every doc. Newest first. Each entry: decision · why · consequence. **Strategic forks live in [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md).**

## ADR-018 — Multi-group membership + Instagram-style switching (2026-06)
**Decision:** One person, many groups; **one `Member` record per (person, group)** with a per-group profile (name/avatar); the app has a **current active group** and a **group switcher** (like Instagram account switching); **no data crosses groups**. See [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D2, [`DATA-MODEL.md`](DATA-MODEL.md) §3.
**Why:** Natural fit for multi-tenant private groups; keeps each group a private island.
**Consequence:** UI has a group switcher + active-group context; identity is shared but profiles are per-group.

## ADR-017 — iOS-only; Android not pursued (2026-06)
**Decision:** Definitively **iOS-only**. No Skip/Android investment. Keep the **thin seam** (core/model import no SwiftUI/CloudKit) but only for **testability + the CRDT Plan-B swap**, not portability. See [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D3.
**Why:** The crew is all-iPhone; cross-platform adds cost for no v1 benefit (SwiftUI doesn't run natively on Android).
**Consequence:** [`TECH-STACK.md`](TECH-STACK.md) cross-platform sections are "considered, not pursued."

## ADR-016 — Multi-tenant of PRIVATE groups (not a public platform) (2026-06)
**Decision:** Any group can create their **own private, invite-only group**, but there's **no public feed/discovery/store/stranger-mixing** in v1. High-trust *within* each group; the public-platform machinery (UGC moderation, age-gating, gallery) is **deferred/out-of-scope** unless we ever add a public surface. See [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D1.
**Why:** Resolves the niche-vs-mass contradiction — adoptable by any crew, without public-platform complexity or cost. Keeps the "for your people" soul.
**Consequence:** Two-trust-zones collapses to ~one (private group + invite edge); free tiers stay sufficient; [`SECURITY.md`](SECURITY.md) §9 and [`PIVOT.md`](PIVOT.md) public bits are deferred.

## ADR-015 — Strategic positioning was OPEN; now resolved by ADR-016/017/018 (2026-06)
**Decision (historical):** We surfaced niche-vs-mass, Android, and monetization as explicit open decisions for the crew rather than burying them. **Now resolved:** see ADR-016 (private multi-tenant), ADR-017 (iOS-only), ADR-018 (switching); only monetization stays deferred ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) O3).
**Why:** An external critique correctly flagged strategic ambiguity as the top risk; surfacing the forks let us decide them cleanly.
**Consequence:** `PIVOT.md` public-platform framing and `TECH-STACK.md` portability are now **deferred / not pursued**, not just "proposed."

## ADR-014 — One model = semantically unified, physically separated (2026-06)
**Decision:** Conceptually one object graph, but `View`/`Reaction`/`Theme` are **independent bounded modules behind interfaces** — not one mega-renderer. Unification lives in envelope/storage/sync/validation only. See [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L2.
**Why:** Avoids the Spotify-HubFramework coupling/performance trap a critique flagged, while keeping the conceptual elegance.
**Consequence:** [`DATA-MODEL.md`](DATA-MODEL.md)/[`ARCHITECTURE.md`](ARCHITECTURE.md) treat the three as separate implementations over one model.

## ADR-013 — Settings/editor-first; AI is a later connector (2026-06)
**Decision:** v1 customization is a **settings/creator surface** (options + token editing + paste scripts/config), **not** conversation-first AI. The only AI work now is **futureproofing** (validated declarative documents = schema-as-API). See [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L1.
**Why:** Conversation-first UX + on-device-AI capability are unvalidated; don't bet v1 on them. The data-not-code design already makes AI a later "just another client."
**Consequence:** [`VISION.md`](VISION.md) Inversion 3 reframed; [`AI-READINESS.md`](AI-READINESS.md) is a futureproofing spec, not a v1 feature; [`CREATOR.md`](CREATOR.md) Script depth is the v1 power surface.

## ADR-012 — Motion & designs are declarative, portable bundles (2026-06, proposed)
**Decision:** Motion/animation is customizable as **data** (motion tokens + Lottie/Rive-style declarative files), never code. A whole look exports/imports as an open **bundle** via the iOS Files app / links. A public store, if wanted, can be **third-party** — we don't run one. See [`CREATOR.md`](CREATOR.md) §11.
**Why:** Full expressive customization while staying App-Store-safe (2.5.2) and portable; sharing without us hosting a store.
**Consequence:** Open bundle format + schema validation + review-before-run for any rules inside; assets validated and slim.

## ADR-011 — One domain-typed object model (2026-06, proposed)
**Decision:** Unify the plumbing around a single reactive object graph; views/reactions/looks are facets of domain-typed objects — not three engines, not generic primitives. (Refined by ADR-014: physically separated modules.)
**Why:** Fewer concepts, ideal substrate for AI, composability; domain typing avoids the HubFramework trap.
**Consequence:** [`SDUI-SPEC.md`](SDUI-SPEC.md) = view facet, [`RULES-SPEC.md`](RULES-SPEC.md) = reaction facet, over one store.

## ADR-010 — Local-first, CloudKit as transport (2026-06, proposed)
**Decision:** Local-first event-sourced/CRDT store on each device; **CloudKit = transport, not source of truth**; R2 holds blobs. **Plan B if CRDT proves too hard:** CloudKit as source of truth + optimistic updates ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3).
**Why:** Slim/fast/smooth structurally, resilience, ownership; sync is a swappable port.
**Consequence:** Biggest new engineering ([`CRITIQUE.md`](CRITIQUE.md) §1.2–1.3); use a proven CRDT library; bound the log with snapshots/compaction.

## ADR-009 — AI-ready by construction (2026-06)
**Decision:** Customization documents are the AI's API; an AI assistant is just another client of the validated propose→preview→approve pipeline; on-device first; external opt-in with consent; actions as App Intents. (Per ADR-013, AI itself is a *later* feature; this is futureproofing.)
**Why:** Data-not-code already makes content AI-legible; baking in schemas now avoids a costly retrofit.
**Consequence:** Schema-first edit API + App Intents pulled in early; AI output re-validated; external AI needs 5.1.2(i) consent.

## ADR-008 — Performance is a gate, not a phase (2026-06)
**Decision:** Every PR meets the budgets in [`PERFORMANCE.md`](PERFORMANCE.md); customization compiles once into typed Swift; hot path is plain SwiftUI.
**Why:** Deep customization is the main jank/bloat risk.
**Consequence:** Perf tests in CI; budgets in Definition of Done.

## ADR-007 — De-risk with spikes before committing (2026-06)
**Decision:** Prototype the risky unknowns (local-first merge, SDUI perf, rules dedup, fallback) before full implementation. See [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) §2, [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3.
**Why:** The engines are the only novel parts; discovering a blocker late kills velocity.
**Consequence:** Spikes run first; nothing built until they pass.

## ADR-006 — Public release brings UGC duties (2026-06, deferred)
**Decision:** *If* we ever add a public surface, implement Guideline 1.2/1.2.1 (EULA, report, block, moderation, age-gating). **Deferred** — v1 is private-groups-only (ADR-016).
**Why:** Required for hosting public user content; not applicable while private.
**Consequence:** No moderation workstream in v1.

## ADR-005 — Customization is data, not code (2026-06)
**Decision:** Layouts and rules are declarative documents interpreted by engines in the binary; never downloaded/executed code, never an HTML5/JS mini-app.
**Why:** App Store 2.5.2 forbids downloaded executable code; 4.7 (mini-apps) adds a regime we don't want.
**Consequence:** Fixed component registry + fixed action allow-list as safety boundaries.

## ADR-004 — Roles are app-level RBAC, light inside a group (2026-06)
**Decision:** `owner`/`admin`/`member`/`invited` enforced in the data layer, not just UI — but kept **light/social** inside a private group (ADR-016, [`VISION.md`](VISION.md) Inversion 4).
**Why:** High-trust private groups don't need an enterprise matrix.
**Consequence:** A roles record per group; every privileged action re-checked.

## ADR-003 — One zone per group (multi-tenant, private) (2026-06)
**Decision:** Each group is its own CloudKit zone; a user participates in many (ADR-016/018). WAD?FC is seeded via the normal create/invite flow, never hardcoded.
**Why:** Serves any private friend group; "membership = the one zone" no longer works.
**Consequence:** Cost rides each user's iCloud quota; the zone boundary is the security wall.

## ADR-002 — Customization = views + reactions + looks (2026-06)
**Decision:** Users customize layouts and behavior, not just colors/fonts. (Reframed by ADR-011/014 as separated facets of one model.)
**Why:** "Every group has special needs and aesthetics."
**Consequence:** Screens/behavior become versioned data with graceful fallback.

## ADR-001 — iOS-only (2026-06)
**Decision:** iOS-only (ADR-017). Keep customization *documents* platform-neutral regardless (cheap, and good hygiene).
**Why:** Deep Apple integration + speed; the crew is all-iPhone.
**Consequence:** CloudKit-centric backend; no cross-platform work.
