# Architecture (brief)

> **Revised for the local-first direction (June 2026).** See [`VISION.md`](VISION.md) for the *why* and [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) for the locked calls (iOS-only, private multi-tenant, group switching). The two customization engines have specs in [`SDUI-SPEC.md`](SDUI-SPEC.md) (views) and [`RULES-SPEC.md`](RULES-SPEC.md) (reactions); the data model in [`DATA-MODEL.md`](DATA-MODEL.md); feasibility/cost in [`FEASIBILITY.md`](FEASIBILITY.md).

## Principles
- **iOS-only** ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D3 — Android not pursued). Go all-in on Apple's stack. A **thin seam** (core/model import no SwiftUI/CloudKit) is kept only for testability + the CRDT Plan-B swap, not portability.
- **Multi-tenant of *private* groups** (D1): any crew creates their own invite-only group; **no public feed/store/strangers** in v1. A person belongs to many groups and **switches the active group like Instagram accounts** (D2), each its own private zone.
- **Local-first.** The group's state lives on each device and merges conflict-free; the cloud is **transport**, not the source of truth. Instant + offline by default. ([`REALTIME.md`](REALTIME.md))
- **One domain-typed model.** Views, reactions, and looks are facets of one object graph — *semantically unified, physically separated* ([`DATA-MODEL.md`](DATA-MODEL.md)).
- **Data, not code.** All customization is declarative data interpreted by engines in the binary — App Store 2.5.2 safe.
- **Settings/editor-first customization;** AI is a *later* client of the same validated edit API ([`AI-READINESS.md`](AI-READINESS.md), [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L1).
- **Non-negotiable, every phase:** privacy by default, **no ads, ever**; and **~$0 hosting**.

## The stack, in layers (bottom → top)

| Layer | What | Built on |
|---|---|---|
| 1. Identity | One person, **many groups** (switchable) | **Sign in with Apple** + a `Member` per (person, group) |
| 2. **Local-first store** | Source of truth: an event-sourced / CRDT log per group on each device; instant, offline, conflict-free; snapshots + compaction bound growth | pure-Swift core + a proven CRDT library |
| 3. **Transport & blobs** | Moves changes + stores heavy files — **not the brain** | **CloudKit** (sync + push, one zone per group) · **R2** (media/assets) |
| 4. **One object model** | Domain-typed objects (`Event`, `Poll`, `Message`, `Tradition`…) with **views/reactions/looks** as *physically separated* facets | the core |
| 5. Engines (in the binary) | **View renderer**, **reaction evaluator**, **motion** — compile-once for speed | Swift interpreters; only *data* is downloaded |
| 6. Creator + AI | The **Creator** (settings/editor-first); AI is a *later* client of the same validated edit API | [`CREATOR.md`](CREATOR.md) · [`AI-READINESS.md`](AI-READINESS.md) |
| 7. Experience | Default app, the **group switcher**, design import/export, Apple integration | SwiftUI, **App Intents/Siri**, Widgets, on-device Foundation Models |

## The thin seam
The brain — model, view compiler, reaction evaluator, validation, catalogs, RBAC — is **pure Swift with no SwiftUI and no CloudKit**, talking out through a few protocols (`Store`, `SyncEngine`, `BlobStore`, `Push`). Kept for **testability + the CRDT Plan-B swap**, not Android ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D3).

## Private, invite-only groups
Each group is its own CloudKit zone; membership = participation in that zone (the security wall). Permissions stay **light inside a group** (high-trust); the public-platform machinery (moderation, gallery, age-gating) is **deferred** unless we ever add a public surface ([`SECURITY.md`](SECURITY.md) §9, [`PIVOT.md`](PIVOT.md)). A person switches between their groups; no data crosses them.

## Free hosting (≈ $0)
CloudKit (data/sync/push, rides each user's iCloud quota) + R2 (10 GB, zero egress) + a free serverless cron + Sign in with Apple. Only recurring cost: **Apple Developer ~$99/yr**. Private + small groups keep us comfortably inside free tiers ([`FEASIBILITY.md`](FEASIBILITY.md)).

## Data-flow notes
- **Instant local, near-real-time remote** (~1–2s via CloudKit push). Two lanes: **durable** state in the synced log; **ephemeral** signals (typing/presence) over a fast lossy channel. ([`REALTIME.md`](REALTIME.md))
- **Reactions are deduplicated** so an automation runs once across devices ([`RULES-SPEC.md`](RULES-SPEC.md) §7).
- **Customization degrades gracefully:** unknown components/reactions never crash an older app; every screen has a built-in default.
- **Plan B:** if CRDT-over-CloudKit proves too hard, CloudKit becomes source of truth + optimistic updates ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3).
- **Performance is a gate** ([`PERFORMANCE.md`](PERFORMANCE.md)).

## Cross-platform (not pursued)
iOS-only (D3). The local-first/thin-seam design *would* make a future port cheaper, but Android is **not pursued** — noted only so the seam isn't mistaken for active cross-platform work.
