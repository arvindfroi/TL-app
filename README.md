# Trivselslederne (TL) app — "WAD?FC"

A **local-first, deeply customizable space for a friend group** — less an app you configure, more a clubhouse you and your friends keep making your own. Plan hangouts, chat, and run the yearly **Trivselslekene**. Our crew (WAD?FC) is the first group.

> **Status: idea / pre-development (June 2026).** No app code yet — this repo is the thinking. The direction has evolved from "a customizable multi-tenant app" to a **local-first, single-model, malleable "group OS"**. **Read [`docs/VISION.md`](docs/VISION.md) first**, then the [`docs/`](docs/) index.

## What we're building (the three pillars)

| Pillar | Feels like | Free on |
|---|---|---|
| **Customizable UI** — themes, layouts, **motion/animation**, icons, fonts; bring-your-own assets; import/export designs | **Cocoon Shell** (theme builder + live preview + asset packs + shareable bundles) | CloudKit (docs) + R2 (assets) |
| **Rules / automation** — trigger → condition → action, friendly builder + power-user scripts | **Minecraft command blocks** | CloudKit + free cron |
| **Real-time chat** — instant, offline-proof, ephemeral media | **Snapchat** | CloudKit push + R2 |

All customization is **declarative data, never code** — which keeps it App-Store-safe (Guideline 2.5.2), fast, AI-readable, and portable. It's **React-like components + CSS-like styling**, rendered natively (see [`docs/TECH-STACK.md`](docs/TECH-STACK.md)).

## The crew (first group)
Arvind · Ruben · Fridrik · Morten · Adrian · Fredrik · Emil · Lars · Eivind

## How it works, in brief
- **Local-first:** the group's state lives on each phone and merges conflict-free; the cloud is **transport**. Instant, works offline, you own your history. ([`docs/REALTIME.md`](docs/REALTIME.md))
- **One model:** layout (*views*), rules (*reactions*), and look (*theme/motion*) are facets of one domain-typed object graph — not three engines. ([`docs/VISION.md`](docs/VISION.md))
- **AI-native:** reshape your space by *asking*, on-device and private; the AI proposes validated changes you approve. ([`docs/AI-READINESS.md`](docs/AI-READINESS.md))
- **Native + portable core:** Swift/SwiftUI for speed + deep iPhone integration (App Intents/Siri, Widgets, Foundation Models); the brain is framework-agnostic behind ports, so Android stays possible. ([`docs/TECH-STACK.md`](docs/TECH-STACK.md))
- **~$0 hosting:** CloudKit + Cloudflare R2 + free serverless cron; only real cost is Apple Developer (~$99/yr). ([`docs/FEASIBILITY.md`](docs/FEASIBILITY.md))
- **Non-negotiable:** privacy by default, **no ads, ever.**

## Documentation
**Start here:** [`docs/VISION.md`](docs/VISION.md) → [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) → [`docs/CREATOR.md`](docs/CREATOR.md) → [`docs/FEASIBILITY.md`](docs/FEASIBILITY.md).
The full, grouped map with reading order is in **[`docs/README.md`](docs/README.md)**. Decisions are logged in [`docs/DECISIONS.md`](docs/DECISIONS.md); honest risks in [`docs/CRITIQUE.md`](docs/CRITIQUE.md). For AI assistants & contributors: [`AGENTS.md`](AGENTS.md).

## Approach
Ruthless focus: **build the spine, ship the heart.** v1 = a beautiful default, fast local-first chat, one ritual, theme/motion customization with live preview, and AI as the friendly on-ramp. Everything else earns its way in. ([`docs/CRITIQUE.md`](docs/CRITIQUE.md) §6)

## Distribution
**TestFlight** for the crew in early phases; a public release (with the App Store UGC duties) is a later, separate bet. Apple Developer Program (~$99/yr) is the main recurring cost.
