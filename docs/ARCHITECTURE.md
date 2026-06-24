# Architecture (brief)

> **Revised for the local-first direction (June 2026).** See [`VISION.md`](VISION.md) for the *why* (this evolves the earlier [`PIVOT.md`](PIVOT.md)). The two customization engines have specs in [`SDUI-SPEC.md`](SDUI-SPEC.md) (views) and [`RULES-SPEC.md`](RULES-SPEC.md) (reactions); feasibility/cost in [`FEASIBILITY.md`](FEASIBILITY.md). This is the orientation summary.

## Principles
- **Local-first.** The group's state lives on each device and merges conflict-free; the cloud is **transport**, not the source of truth. Instant + offline by default. ([`REALTIME.md`](REALTIME.md))
- **One domain-typed model.** Views (layout), reactions (rules), and looks (theme/motion) are facets of one object graph — not three engines, not generic primitives. ([`VISION.md`](VISION.md) Inversion 2)
- **Data, not code.** All customization (views, reactions, themes, motion) is declarative data interpreted by engines in the binary — App Store 2.5.2 safe. ([`AI-READINESS.md`](AI-READINESS.md))
- **Multi-tenant, high-trust inside / careful between.** Many groups; warm and simple inside a group, guarded at the edges (imports, any public sharing). ([`VISION.md`](VISION.md) Inversion 4)
- **iOS now, portable core.** Native Swift/SwiftUI for speed + Apple integration; the brain is framework-agnostic behind ports so Android/longevity stay open. ([`TECH-STACK.md`](TECH-STACK.md))
- **Non-negotiable, every phase:** privacy by default, **no ads, ever**; and **~$0 hosting**.

## The stack, in layers (bottom → top)

| Layer | What | Built on |
|---|---|---|
| 1. Identity | One person, many groups | **Sign in with Apple** |
| 2. **Local-first store** | The source of truth: an event-sourced / CRDT log on each device; instant, offline, conflict-free merge; snapshots + compaction bound its growth | pure-Swift core + a proven CRDT library |
| 3. **Transport & blobs** | Moves changes between devices and stores heavy files — **not the brain** | **CloudKit** (sync + push) · **Cloudflare R2** (media/assets, zero egress) |
| 4. **One object model** | Domain-typed objects (`Event`, `Poll`, `Message`, `Tradition`, `Member`…) with **views**, **reactions**, and **looks** as facets | the core |
| 5. Engines (in the binary) | **View renderer** (data→native SwiftUI), **reaction evaluator** (automation), **motion** (declarative) — compile-once for speed | Swift interpreters; only *data* is downloaded |
| 6. Creator + AI | The **Creator** (visual + node + script + console) and the **AI assistant** — both clients of one validated propose→preview→approve edit API | [`CREATOR.md`](CREATOR.md) · [`AI-READINESS.md`](AI-READINESS.md) |
| 7. Integration | Default app, design import/export (Files), Apple integration | SwiftUI, **App Intents/Siri**, Widgets, on-device Foundation Models |

## The portable core (ports & adapters)
The brain — model, view compiler, reaction evaluator, validation, catalogs, RBAC/policy — is **pure Swift with no SwiftUI and no CloudKit**. Everything Apple-specific sits behind protocols (`Store`, `SyncEngine`, `BlobStore`, `Push`, `AIProvider`) with adapters (CloudKit, R2, APNs, Foundation Models). Consequence: getting faster = improve an adapter; swapping the backend = one new adapter; **Android via Skip** = reuse the core + a second renderer. ([`TECH-STACK.md`](TECH-STACK.md) §3)

## Two trust zones
- **Inside a group:** ~9 friends who trust each other → light, social permissions, generous undo, minimal machinery. ([`VISION.md`](VISION.md) Inversion 4)
- **At the edges:** imported design bundles and any cross-group/public sharing are **validated-as-data, reviewed-before-run**, and (if ever public) carry the UGC duties from [`PIVOT.md`](PIVOT.md) §6. A public store can be **third-party** — we needn't run one ([`CREATOR.md`](CREATOR.md) §11).

## Free hosting (≈ $0)
CloudKit (data/sync/push, rides each user's iCloud quota) + R2 (10 GB, zero egress) + a free serverless cron (scheduled reactions) + Sign in with Apple. Only recurring cost: **Apple Developer ~$99/yr**. The two deferrable edges that can cost a little: crisp always-on presence and a self-run public store — both optional. ([`FEASIBILITY.md`](FEASIBILITY.md))

## Data-flow notes
- **Instant local, near-real-time remote.** Your action applies immediately on-device; CloudKit push propagates to others in ~1–2s. Two lanes: **durable** state (messages/events/polls/points) in the synced log; **ephemeral** signals (typing/presence) over a fast lossy channel, never stored. ([`REALTIME.md`](REALTIME.md))
- **Reactions are deduplicated** so an automation runs once across all devices, not per device ([`RULES-SPEC.md`](RULES-SPEC.md) §7).
- **Customization degrades gracefully:** unknown components/reactions from a newer version never crash an older one; every screen has a built-in default ([`SDUI-SPEC.md`](SDUI-SPEC.md) §7).
- **Performance is a gate** ([`PERFORMANCE.md`](PERFORMANCE.md)): compile-once, lazy render, off-main reactions, motion within a frame budget.

## Cross-platform fallback (not active)
Android later = a second renderer/evaluator over the same model + a non-Apple `SyncEngine` adapter (Skip, or Firebase). The CRDT/local-first design makes this far cheaper than a CloudKit-welded app would. Not active while iOS-only.
