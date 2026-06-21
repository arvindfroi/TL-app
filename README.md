# Trivselslederne (TL) app — "WAD?FC"

A private, invite-only iPhone app for one friend group of 9. It does three things well: **plan hangouts**, **group chat**, and run our yearly event **Trivselslekene**.

> Status: **planning / pre-development.** This repo currently holds the product and architecture docs. Code comes next.

## 🤖 Working on this with an AI assistant?
Point it at **[`AGENTS.md`](AGENTS.md)** — a self-contained guide that explains the project, where the truth lives, how work is tracked, and the rules to follow. Humans should read **[`CONTRIBUTING.md`](CONTRIBUTING.md)**.

## Principles (whole project)
- **Security & privacy by default** — closed group, Sign in with Apple, minimal data, encrypted.
- **No ads, ever** — no ad SDKs, no tracking-for-ads, no sponsored content.

## The crew
Arvind · Ruben · Fridrik · Morten · Adrian · Fredrik · Emil · Lars · Eivind

## What it is
- **iOS-only** (Swift / SwiftUI), deep iPhone integration (Widgets, Live Activities, Siri, Apple Intelligence).
- **$0 hosting**: Apple **CloudKit** for app data + **Cloudflare R2** for photo/video. No server to run.
- **Closed group**: membership is a CloudKit shared zone — no public sign-up.

## MVP (v1.0)
Group chat with read receipts · Events + RSVP · Polls · "Hjemme i Sandnes" calendar · Push notifications.

See the full plan in [`docs/MVP-and-Roadmap.md`](docs/MVP-and-Roadmap.md).

## Progress
Live progress by phase is in **[`STATUS.md`](STATUS.md)** (auto-generated from Issues). Per-person activity is under the repo's **Insights → Contributors / Pulse**.

## Roadmap at a glance
- **Phase 0** — Foundations (Apple Developer, CloudKit container + shared zone, app scaffold)
- **Phase 1** — MVP core loop
- **Phase 2** — Social (daily vlog / BeReal, wishlists + Secret Santa, bucket list) + iOS extras
- **Phase 3** — Trivselslekene & gamification (leaderboard, achievements)
- **Phase 4** — Money features (TL ferien 2030 savings, Vipps "polymarket" — points-only first)

## Tech stack
Swift / SwiftUI · Sign in with Apple · CloudKit (+ CKShare) · CloudKit subscriptions for push · Cloudflare R2 for media · free serverless cron (Cloudflare Workers / Val Town) · Apple Intelligence / Foundation Models (opportunistic).

## Docs
- [`AGENTS.md`](AGENTS.md) — AI assistant & contributor guide (start here)
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — human contributor workflow
- [`DESIGN.md`](DESIGN.md) — design language (draft) + decisions checklist
- [`STATUS.md`](STATUS.md) — auto-generated progress
- [`docs/MVP-and-Roadmap.md`](docs/MVP-and-Roadmap.md) — full MVP, hosting decision, roadmap, risks
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — the CloudKit + R2 architecture in brief
- [`docs/BACKLOG.md`](docs/BACKLOG.md) — feature backlog (seed for GitHub Issues)
- [`docs/brainstorm.md`](docs/brainstorm.md) — the original idea dump

## Distribution
TestFlight-only for the group (avoids App Store user-generated-content review). Apple Developer Program (~$99/yr) is the only recurring cost.
