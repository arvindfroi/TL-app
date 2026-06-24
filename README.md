# Trivselslederne (TL) app — "WAD?FC"

A **customizable** iPhone app for organizing friend groups and hosting events. Each group makes it their own — their **layouts, design, and automation rules**, not just colors. Our crew (WAD?FC) is the first group and design partner.

> **Status: planning / pre-development — pivoting (June 2026).** This repo holds product + architecture docs. We are mid-pivot from "one private app for nine friends" to "a platform any friend group can adopt and reshape." Read [`docs/PIVOT.md`](docs/PIVOT.md) first.

## The pivot in one line
From a fixed app for one hardcoded group → **a multi-tenant app with admins, invites, and deep per-group customization** (user-editable layouts via a server-driven-UI engine, plus a sandboxed rules/automation engine for power users). Still **Swift/SwiftUI**, still **iOS-only for now**, still **ad-free and privacy-first**.

## The crew (first group)
Arvind · Ruben · Fridrik · Morten · Adrian · Fredrik · Emil · Lars · Eivind

## What it is
- **iOS-only** (Swift / SwiftUI), deep iPhone integration (Widgets, Live Activities, Siri, Apple Intelligence).
- **Multi-tenant:** any friend group can create a group, invite members, assign **admins**, and customize.
- **Deeply customizable:** editable **layouts** (SDUI) + **theme tokens** + a **rules/automation engine** — "advanced settings" for power users, friendly editors for everyone.
- **$0-leaning hosting:** Apple **CloudKit** (one CKShare zone per group) + **Cloudflare R2** for media. Cost stays near-zero because it rides each user's iCloud quota — which is what keeps it **ad-free** as a public app.
- **Data, not code:** customization is declarative data interpreted by engines in the binary — App Store **2.5.2** safe (and deliberately not the HTML5/JS mini-app route, Guideline 4.7).

## Core jobs (ship as the default template)
Plan hangouts · group chat · run the yearly **Trivselslekene**. These become the default, restylable template every group starts from.

## Roadmap at a glance (re-cut for the pivot)
- **Phase 0** — Platform foundations: multi-group data model, **roles & invites**, CloudKit, app scaffold
- **Phase 1** — Theming + the core loop (chat, events/RSVP, polls, calendar, push) built on theme tokens — **the real launch**
- **Phase 2** — SDUI **layout engine** + friendly editor (groups change layouts, not just styles)
- **Phase 3** — **Rules / automation engine** ("advanced settings")
- **Phase 4** — Public release: UGC moderation (Guideline 1.2) + **template gallery**
- **Phase 5** — Trivselslekene, gamification, iOS extras, money (with care) as editable modules

## Tech stack
Swift / SwiftUI · Sign in with Apple · CloudKit (CKShare zone per group) + app-level RBAC · CloudKit subscriptions for push · Cloudflare R2 for media · free serverless cron (Workers / Val Town) for scheduled rules · Apple Intelligence / Foundation Models (opportunistic). *Cross-platform later: a second renderer over the same documents via Skip, or Firebase Spark.*

## Docs
- [`docs/PIVOT.md`](docs/PIVOT.md) — **start here:** the pivot plan (scope, new architecture, risks, roadmap, migration)
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — the seven-layer multi-tenant architecture
- [`docs/SDUI-SPEC.md`](docs/SDUI-SPEC.md) — the customizable layout engine
- [`docs/RULES-SPEC.md`](docs/RULES-SPEC.md) — the rules / automation engine
- [`docs/PERFORMANCE.md`](docs/PERFORMANCE.md) — how customization stays **slim, fast, smooth** (budgets + rules)
- [`docs/EXECUTION-PLAN.md`](docs/EXECUTION-PLAN.md) — spikes, critical path, Definition of Ready/Done, CI
- [`docs/DECISIONS.md`](docs/DECISIONS.md) — architecture decision log (the *why*, newest first)
- [`docs/MVP-and-Roadmap.md`](docs/MVP-and-Roadmap.md) — full roadmap, hosting, risks
- [`docs/BACKLOG.md`](docs/BACKLOG.md) — feature backlog (seed for Issues)
- [`DESIGN.md`](DESIGN.md) — design language → now the *default theme*
- [`docs/brainstorm.md`](docs/brainstorm.md) — original idea dump

## Distribution
Pivot adds a **public App Store release** (Phase 4) — which brings Guideline 1.2 UGC duties (EULA, report/block, moderation, age-gating). Early phases stay **TestFlight** for our group. Apple Developer Program (~$99/yr) is the main recurring cost.
