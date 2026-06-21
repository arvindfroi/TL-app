# Trivselslederne (TL) app — "WAD?FC"

A private, invite-only iPhone app for one friend group of 9. It does three things well: **plan hangouts**, **group chat**, and run our yearly event **Trivselslekene**.

> Status: **planning / pre-development.** This repo currently holds the product and architecture docs. Code comes next.

## The crew
Arvind · Ruben · Fridrik · Morten · Adrian · Fredrik · Emil · Lars · Eivind

## What it is
- **iOS-only** (Swift / SwiftUI), deep iPhone integration (Widgets, Live Activities, Siri, Apple Intelligence).
- **$0 hosting**: Apple **CloudKit** for app data + **Cloudflare R2** for photo/video. No server to run.
- **Closed group**: membership is a CloudKit shared zone — no public sign-up.

## MVP (v1.0)
Group chat with read receipts · Events + RSVP · Polls · "Hjemme i Sandnes" calendar · Push notifications.

See the full plan in [`docs/MVP-and-Roadmap.md`](docs/MVP-and-Roadmap.md).

## Roadmap at a glance
- **Phase 0** — Foundations (Apple Developer, CloudKit container + shared zone, app scaffold)
- **Phase 1** — MVP core loop
- **Phase 2** — Social (daily vlog / BeReal, wishlists + Secret Santa, bucket list) + iOS extras
- **Phase 3** — Trivselslekene & gamification (leaderboard, achievements)
- **Phase 4** — Money features (TL ferien 2030 savings, Vipps "polymarket" — points-only first)

## Tech stack
Swift / SwiftUI · Sign in with Apple · CloudKit (+ CKShare) · CloudKit subscriptions for push · Cloudflare R2 for media · free serverless cron (Cloudflare Workers / Val Town) · Apple Intelligence / Foundation Models (opportunistic).

## Docs
- [`docs/MVP-and-Roadmap.md`](docs/MVP-and-Roadmap.md) — full MVP, hosting decision, roadmap, risks
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — the CloudKit + R2 architecture in brief
- [`docs/BACKLOG.md`](docs/BACKLOG.md) — feature backlog (seed for GitHub Issues)
- [`docs/brainstorm.md`](docs/brainstorm.md) — the original idea dump

## Distribution
TestFlight-only for the group (avoids App Store user-generated-content review). Apple Developer Program (~$99/yr) is the only recurring cost.
