# TL-appen

> **Working codename.** "TL-appen" is a placeholder while the project moves from a single-group app to a product any friend group can use. A real brand name is still to be decided.

A **local-first, deeply customizable app for a friend group**. It ships as a usable **base app** for hanging out — group chat and hangout planning out of the box — but the whole point is that **everything in it can be customized and configured**: the layout, the look, the motion, and the behavior. Less an app you're handed, more a clubhouse you keep remaking.

> **Status: idea / pre-development (June 2026).** No app code yet — this repo is the thinking.

## What it does

Out of the box it's a usable base app for any friend group:

- **Group chat** — one shared space, instant, with read receipts.
- **Plan hangouts** — events, RSVP, polls, and a "who's home" availability calendar.

And **everything in it is customizable and configurable.** Anything more specialized — a yearly games competition with scoring, a shared savings pot, a prediction game, Secret Santa, a bucket list — is **not** a built-in feature. It's something a group builds and configures with the app's own tools (objects, layouts, automations, points, lists), then keeps or shares as a portable template.

Any friend group can create their own **private, invite-only** group — it's multi-tenant, but a network of private islands, not a public social platform.

## How it's built (in brief)

- **Local-first:** the group's state lives on each phone and merges conflict-free; the cloud is just transport. Instant, works offline, you own your history.
- **iOS-native:** Swift / SwiftUI, deep iPhone integration (App Intents/Siri, Widgets, on-device AI). iOS-only — Android not pursued.
- **Customize anything, safely:** layout, design, motion, and behavior are all **declarative data, never code** — editable, scriptable, and importable/exportable (App Store-safe). It's React-like + CSS-like, rendered natively.
- **~$0 hosting:** CloudKit + Cloudflare R2 + free serverless cron. Only real cost is Apple Developer (~$99/yr).
- **Non-negotiable:** privacy by default, **no ads, ever.**

## The plan

Everything — vision, locked decisions, architecture, data model, the customization "Creator", cost, roadmap, and risks — is in **[`PLAN.md`](PLAN.md)**. That's the single source of truth; start there.

Older detailed working docs (engine specs, prior-art notes, the original brainstorm, the full decision log) are kept in a local `archive/` folder for reference — not part of this repo. They're superseded; when in doubt, `PLAN.md` wins.
