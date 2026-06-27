# Trivselslederne (TL) app — WAD?FC

A **local-first, deeply customizable space for a friend group** — less an app you configure, more a clubhouse you and your friends keep making your own. Plan hangouts, chat, and run the yearly **Trivselslekene**.

> **Status: idea / pre-development (June 2026).** No app code yet — this repo is the thinking. Our crew (WAD?FC) is the first group.

## What it does

- **Plan hangouts** — events, RSVP, polls, and a "who's home in Sandnes" calendar.
- **Group chat** — one shared space, instant, with read receipts.
- **Trivselslekene** — our yearly games event: scoring, leaderboard, trophies, history.

Other friend groups can create their own **private, invite-only** group too — it's multi-tenant, but a network of private islands, not a public social platform.

## How it's built (in brief)

- **Local-first:** the group's state lives on each phone and merges conflict-free; the cloud is just transport. Instant, works offline, you own your history.
- **iOS-native:** Swift / SwiftUI, deep iPhone integration (App Intents/Siri, Widgets, on-device AI). iOS-only — Android not pursued.
- **Customizable, safely:** every group can reshape its layouts, design, motion, and automations — all as **declarative data, never code** (App Store-safe). It's React-like + CSS-like, rendered natively.
- **~$0 hosting:** CloudKit + Cloudflare R2 + free serverless cron. Only real cost is Apple Developer (~$99/yr).
- **Non-negotiable:** privacy by default, **no ads, ever.**

## The crew (first group)

Arvind · Ruben · Fridrik · Morten · Adrian · Fredrik · Emil · Lars · Eivind

## The plan

Everything — vision, locked decisions, architecture, data model, the customization "Creator", cost, roadmap, and risks — is in **[`PLAN.md`](PLAN.md)**. That's the single source of truth; start there.

Older detailed working docs (engine specs, prior-art notes, the original brainstorm, the full decision log) are kept off the repo as an archive for reference. They're superseded — when in doubt, `PLAN.md` wins.
