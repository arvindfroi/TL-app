# Architecture (brief)

> **Revised for the customization pivot (June 2026).** See [`PIVOT.md`](PIVOT.md) for the why; the two engines have full specs in [`SDUI-SPEC.md`](SDUI-SPEC.md) and [`RULES-SPEC.md`](RULES-SPEC.md). This is the orientation summary.

## Principles
- **iOS-only (for now).** Go all-in on Apple's stack; keep the customization engines platform-neutral in *data* so Android (Skip) stays possible later without a format change.
- **No server to run.** Managed free tiers; tiny serverless functions only where scheduled logic truly needs them.
- **Multi-tenant by design.** Many independent groups, each with roles and invites. Our crew (WAD?FC) is the first group, created through the same flow everyone uses — **never hardcoded**.
- **Data, not code.** Customization (layouts + rules) is declarative data interpreted by engines baked into the binary — App Store 2.5.2 safe.
- **Non-negotiable, every phase:** privacy by default, and **no ads, ever.**

## The seven layers

| Layer | What | Built on |
|---|---|---|
| 1. Identity | One person, many groups | **Sign in with Apple** |
| 2. Tenancy & access | Each **group** = one shared zone; you participate in every group you're in | **CloudKit CKShare** (the hard security wall) |
| 3. Roles & permissions | `owner` / `admin` / `member` / `invited`; who can edit layouts, write rules, moderate | App-level **RBAC** records, enforced in the data layer |
| 4. Domain data | Messages, events, RSVPs, polls, availability, points; media *metadata* | **CloudKit**; media files on **Cloudflare R2** |
| 5. Customization store | Per-group theme tokens, layout documents, rule definitions (versioned, with fallback) | **CloudKit** (in the group zone) |
| 6. Engines (in the binary) | **SDUI renderer** (data → native SwiftUI) and **Rules evaluator** (declarative automation) | Swift interpreters; only *data* is downloaded |
| 7. Experience & integration | Default app, the customization **editor** (friendly + advanced), **template gallery**, Apple integration | SwiftUI, Widgets, Live Activities, Siri |

## Why CloudKit still fits a multi-tenant app
A user can **participate in many CKShares at once**, so "each group is its own shared zone" scales to many groups per person without a backend. Crucially, CloudKit storage/traffic sits largely in **each user's own iCloud quota**, so per-tenant cost stays near-zero as the number of groups grows — this is what keeps a public, **ad-free** app sustainable. CKShare gives *access* (in/out of a zone); **roles within a group are an app-level concept** (layer 3) because CKShare only models participant read/write, not admin/moderator semantics.

## Components

| Concern | Choice | Why |
|---|---|---|
| Identity | **Sign in with Apple** | No passwords stored |
| Per-group data + access | **CloudKit CKShare zone per group** | Free, zone boundary = security wall, many shares per user |
| Roles / permissions | **App-level RBAC records** in the group zone | CKShare can't express admin/moderator roles |
| Push notifications | **CloudKit subscriptions** (CKQuerySubscription) | Free, no server |
| Media files | **Cloudflare R2** | 10 GB free, zero egress; metadata stays in CloudKit |
| Scheduled jobs (rules, "who's it", expiry, R2 signing) | **Cloudflare Workers / Val Town** free tier | Free serverless cron |
| Customization (layouts/themes/rules) | **Declarative documents** in the group zone | Data-not-code (2.5.2); see engine specs |
| On-device AI (optional) | **Foundation Models framework** | Free, private, on-device; gated to capable devices |

## Data-flow notes
- **Sync is near-real-time (seconds), not live co-editing.** Fine for chat/polls/events and for layout/rule edits (last-write-wins via change tokens; edit-lock hint for active layout editing). True simultaneous co-editing remains out of scope.
- **Rule execution is deduplicated** so an automation runs once across all members' devices, not per device — via a claim record + idempotent actions (see [`RULES-SPEC.md`](RULES-SPEC.md) §7).
- **Customization degrades gracefully:** unknown components/rules from a newer app version never crash an older one; every screen has a built-in default layout to fall back to.
- **Vlogs** stay ephemeral: files in R2, metadata + expiry in CloudKit; steady-state storage stays small.
- **Backups:** periodic CloudKit export to Drive.

## New obligations from going public (multi-tenant)
A public release pulls in App Store **Guideline 1.2** (UGC): EULA, in-app **report/block**, moderation/holding of flagged content, and **age-gating (1.2.1)**. Group **admins** are first-line moderators (powers from the RBAC layer); we provide the tooling and a backstop reporting path. Customization must never let a layout/rule cross the zone boundary or escalate privileges — the defenses are the CKShare wall, the sandboxed engines, and validate-and-degrade. See [`PIVOT.md`](PIVOT.md) §6.

## Cross-platform fallback (not active)
If Android is ever needed: build a second **renderer/evaluator over the same documents** (via **Skip**), or fall back to **Firebase (Spark, free)** for a shared backend. Avoid Supabase (free projects pause when idle). Not active while iOS-only.
