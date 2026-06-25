# OPEN-DECISIONS — the strategic forks (to lock with Ruben)

> **Status: planning, no code yet (June 2026).** Prompted by a strong external critique whose core point is right: the biggest risk isn't any one tech choice, it's **strategic ambiguity** — the docs have been trying to be niche *and* mass, free *and* sustainable, simple *and* multi-tenant, experimental *and* shippable. This doc **makes the forks explicit** instead of burying them. A few calls are locked; the big ones wait for **Ruben's input**. When a doc disagrees with this file on these topics, **this file wins** (and [`DECISIONS.md`](DECISIONS.md)).

## Locked now

### L1 — Customization is settings/editor-first; AI is a *later* connector; futureproof now
v1 customization is a **settings / creator surface**: choose options, edit theme/motion tokens with live preview, and (for power users) **paste scripts and config** directly (the "Script" depth in [`CREATOR.md`](CREATOR.md)). **No conversation-first AI in v1.** The *only* AI work now is **futureproofing**: keep all customization as **validated declarative documents** (schema-as-API), so an AI connector later is "just another client" of the same validated edit pipeline ([`AI-READINESS.md`](AI-READINESS.md)). This resolves the critique's "AI-native UX is unvalidated" risk — we don't bet v1 on it.

### L2 — "One model" = *semantically unified, physically separated*
Conceptually it's one object graph ([`VISION.md`](VISION.md) Inversion 2). **In implementation, `View`, `Reaction`, and `Theme` are independent, well-bounded modules behind explicit interfaces — not one mega-renderer.** The unification lives in the *envelope, storage, sync, and validation* ([`DATA-MODEL.md`](DATA-MODEL.md)), not in a single component that does everything. This keeps the conceptual elegance while dodging the Spotify-HubFramework performance/coupling trap the critique (rightly) flags.

### L3 — Spikes before any product code; the CRDT has a documented fallback
Do the four spikes (L/M/N/R) **first** ([`EXECUTION-PLAN.md`](EXECUTION-PLAN.md)). **Spike L's bar is raised** per the critique: not "does sync work?" but *"under concurrent edits + network interruptions + multiple devices, does the merge match user expectations — and can we answer 'why was my edit overwritten?'"* And we name a **Plan B up front**: if CRDT-over-CloudKit proves uncontrollable (composition/tombstone/explainability pain), fall back to **CloudKit as source of truth + optimistic local updates** — we lose pure offline-merge but keep the instant feel. Local-first is the goal, not a religion.

## Open — need Ruben (and the crew)

### O1 — Positioning: us-first vs. multi-tenant public ⭐ *the decision everything hangs on*
The critique's sharpest point. The current docs assert both, which is the core contradiction.

| Dimension | "Us-first, simple" | "Multi-tenant public platform" |
|---|---|---|
| Trust/security | High-trust, light permissions, no moderation | Real RBAC + UGC moderation + age-gating |
| Hosting/cost | Free tiers are plenty for ~9 | Free tiers exhaust; needs a cost/monetization plan |
| Complexity | Much smaller surface; delete the two-trust-zone logic | Context-sensitive security; the hard version |
| Soul (the moat) | Strongest ("for your people") | Diluted toward "another configurable app" |

**Recommendation (matches [`CRITIQUE.md`](CRITIQUE.md) §6):** *us-first and simple*; treat "other groups adopt it" as a happy byproduct and **multi-tenant public as a separate, explicit later bet**. That deletes a large amount of v1 complexity and keeps the soul. **But this is Ruben's call too — holding.** Until decided, treat [`PIVOT.md`](PIVOT.md)'s multi-tenant framing as **proposed, not locked**.

### O2 — Android: how much portability to build now
SwiftUI doesn't run natively on Android (Skip *transpiles* it), so "portable" has a real cost. Three lanes:

- **iOS-only, simple** — drop heavy abstractions, most direct SwiftUI app; revisit Android later as a bigger lift.
- **Thin seam (recommended)** — one cheap discipline: the **data model + core logic import neither SwiftUI nor CloudKit**, talking out through a few protocols (`Store`, `SyncEngine`). A future Android port or backend swap rewrites *adapters*, not the brain. Far less work than full ports/adapters; far more insurance than none; and it's just cleaner, more testable code regardless.
- **Commit to Android post-v1** — invest now in a genuinely portable core (Skip).

**Recommendation:** *thin seam* — it's the only one of the three that's cheap even if Android never happens. **Holding for Ruben.** (This also softens [`TECH-STACK.md`](TECH-STACK.md)'s heavier "ports & adapters" framing toward "thin seam.")

### O3 — Sustainability / monetization: **deferred**
Only becomes a real question if O1 → multi-tenant. While us-first, free tiers are fine and "no ads, ever" stands. Revisit *with* O1.

## How this answers the critique's four contradictions

| Contradiction | Resolution / status |
|---|---|
| Niche vs. Mass | **O1 — open**, recommendation us-first; docs no longer silently assert mass. |
| Free vs. Sustainable | **O3 — deferred**, tied to O1; fine while us-first. |
| Simple vs. Powerful | **Locked toward simple:** L1 (editor-first, AI later), L2 (separated modules), O1-rec (drop two-trust-zone in v1). |
| Experimental vs. Deliverable | **Locked:** L1 (no conversation-first bet) + L3 (spikes-first, CRDT Plan B). |

## What changes in the other docs (applied)
- [`VISION.md`](VISION.md): banner pointing here; Inversion 3 reframed to settings/editor-first with AI later; "one model" notes physical separation.
- [`AI-READINESS.md`](AI-READINESS.md): AI is explicitly **later**; v1 only futureproofs.
- [`DATA-MODEL.md`](DATA-MODEL.md): adds the Plan-B fallback + merge-explainability + physically-separated note.
- [`DECISIONS.md`](DECISIONS.md): ADRs for L1–L3 and the open O1–O3.
- [`PIVOT.md`](PIVOT.md) / [`TECH-STACK.md`](TECH-STACK.md): flagged as *proposed* on positioning/portability pending O1/O2.

---
*Next: Ruben weighs in on O1–O2 (a GitHub discussion issue is open for this). Nothing gets built until the spikes run.*
