# OPEN-DECISIONS — the strategic forks (now mostly resolved)

> **Status: planning, no code yet (June 2026).** A critique flagged **strategic ambiguity** as the top risk. We've now made the calls (Arvind + the crew); only monetization is deferred. When a doc disagrees with this file on these topics, **this file wins** (and [`DECISIONS.md`](DECISIONS.md)).

## Decided

### D1 — Positioning: multi-tenant, but **private groups** (not a public platform)
We build the app **our** group wants — *and* make it so **any group can create their own private group**. The key clarification that dissolves the niche-vs-mass contradiction: this is **multi-tenant of private islands, not a public social network.**
- Groups are **invite-only**. There is **no public feed, no discovery, no stranger-mixing, no public template store** in v1.
- So **high-trust *within* each group stays the model** (light, social permissions). The "two trust zones" essentially collapses to **one** — a private group plus its invite edge.
- **No at-large UGC moderation / age-gating / public gallery in v1** — those only return *if* we ever add a public surface (not planned). [`SECURITY.md`](SECURITY.md) §9 and [`PIVOT.md`](PIVOT.md)'s public-platform machinery are **deferred/out-of-scope** for now.
- **Free tiers stay fine** (each group is small + private), so monetization stays moot — see O3.

This keeps the soul ("for your people") *and* lets other crews adopt it, **without** the public-platform complexity the critique warned about.

### D2 — Multi-group membership + Instagram-style profile switching
A person can belong to **multiple groups** and **switches the active group like switching accounts on Instagram.**
- One **Sign in with Apple** identity; a **per-group member profile** (your display name/avatar can differ per group).
- A **group switcher** in the UI; **one active-group context** at a time.
- Each group is its own **private space** (its own CloudKit zone) with its own customization, chat, members, and history. **No data crosses groups.**
- Notifications can still surface across groups (e.g. a badge on another group), but content stays isolated.

This is a clean fit for the data model ([`DATA-MODEL.md`](DATA-MODEL.md) §3): one person → many `Member` records, one per group; the app holds a "current group" and renders that zone.

### D3 — iOS-only (Android not pursued)
Definitively **iOS-only.** We do **not** invest in Skip/Android. We keep one cheap discipline — the **thin seam** (the data model + core logic import no SwiftUI and no CloudKit) — but **purely for testability and the CRDT Plan-B swap**, *not* for Android. [`TECH-STACK.md`](TECH-STACK.md)'s cross-platform sections are now "considered, not pursued."

### L1–L3 — still locked (engineering)
- **L1** — Customization is **settings/editor-first**; AI is a **later** connector; v1 only *futureproofs* (validated declarative documents). [`AI-READINESS.md`](AI-READINESS.md).
- **L2** — "One model" = **semantically unified, physically separated** (View/Reaction/Theme are independent modules). [`DATA-MODEL.md`](DATA-MODEL.md).
- **L3** — **Spikes before any product code**; **CRDT has a named Plan B** (CloudKit source-of-truth + optimistic) if Spike L fails. [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md).

## Deferred

### O3 — Sustainability / monetization
Private + small + free tiers = fine. **"No ads, ever" stands.** Revisit only if scale ever changes (not planned).

## How this answers the critique's contradictions

| Contradiction | Resolution |
|---|---|
| Niche vs. Mass | **Resolved (D1):** multi-tenant of *private* groups — adoptable by any crew, but not a public platform, so no mass-scale machinery. |
| Free vs. Sustainable | **Deferred (O3):** free is fine for private small groups; no monetization needed. |
| Simple vs. Powerful | **Simple wins where it counts:** D1 (no public layer), D3 (one platform), L1 (editor-first), L2 (separated modules). |
| Experimental vs. Deliverable | **Deliverable:** L1 (no conversation-first bet) + L3 (spikes-first, CRDT Plan B). |

## What changed in the docs (applied)
- [`DECISIONS.md`](DECISIONS.md): ADRs for D1–D3.
- [`DATA-MODEL.md`](DATA-MODEL.md): identity = one person, many groups, per-group profile, active-group context.
- [`VISION.md`](VISION.md) / [`AGENTS.md`](AGENTS.md) / [`ARCHITECTURE.md`](ARCHITECTURE.md): iOS-only; multi-tenant **private**; group switching.
- [`TECH-STACK.md`](TECH-STACK.md): iOS-only (Android not pursued; thin seam = cleanliness).
- [`SECURITY.md`](SECURITY.md): v1 is private-groups-only; public/UGC/gallery machinery deferred.
- [`PIVOT.md`](PIVOT.md): public-platform framing is **deferred**, not v1.

---
*Next: nothing gets built until the spikes ([`EXECUTION-PLAN.md`](EXECUTION-PLAN.md)) run. Ruben can still weigh in (issue #29), but the direction is set.*
