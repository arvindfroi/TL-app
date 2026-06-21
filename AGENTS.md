# AGENTS.md — AI assistant & contributor guide

> **If you are an AI assistant:** read this whole file first. It tells you what this project is, where the truth lives, how work is tracked, and the rules to follow. When you need *current* status (what's done, what's next), read the live sources listed in [Sources of truth](#2-sources-of-truth) — do **not** rely on memory or on what this file says about progress. Prefer proposing changes as a pull request that links the relevant issue.

This file follows the [AGENTS.md](https://agents.md) convention and is auto-detected by most AI coding tools. Humans: see also [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## 1. Project snapshot

- **What:** Trivselslederne ("TL") — a private, invite-only **iPhone** app for one friend group of 9 ("WAD?FC"). It does three things well: **plan hangouts**, **group chat**, and run the yearly event **Trivselslekene**.
- **Who:** Arvind, Ruben, Fridrik, Morten, Adrian, Fredrik, Emil, Lars, Eivind.
- **Platform:** iOS-only (Swift / SwiftUI). Deep iPhone integration is a goal (Widgets, Live Activities, Siri, Apple Intelligence).
- **Backend:** Apple **CloudKit** (app data) + **Cloudflare R2** (media). No server we run. $0 hosting; only cost is the Apple Developer membership (~$99/yr).
- **Status:** **planning / pre-development.** No app code yet — this repo currently holds product + architecture docs and the issue backlog. Code arrives with Phase 0.
- **Non-negotiables (every phase, every PR):** **security & privacy by default**, and **no ads — ever**. See [§8 Guardrails](#8-guardrails-do--dont).

## 2. Sources of truth

Always check these for current reality before acting:

| You want to know… | Look here |
|---|---|
| Current progress / what's done | [`STATUS.md`](STATUS.md) (auto-generated) and the **Issues** tab |
| What to build & in what order | [GitHub Issues](../../issues) filtered by phase label; full plan in [`docs/MVP-and-Roadmap.md`](docs/MVP-and-Roadmap.md) |
| Why the architecture is the way it is | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) and §4 of the roadmap |
| The original feature ideas | [`docs/brainstorm.md`](docs/brainstorm.md) |
| The feature backlog (master list) | [`docs/BACKLOG.md`](docs/BACKLOG.md) (mirrored as Issues) |
| The design language (type, color, motion) | [`DESIGN.md`](DESIGN.md) (currently a draft/scaffold) |
| Who's doing what / activity | GitHub **Insights → Contributors / Pulse**, issue **assignees** |

**Rule for AI assistants:** the roadmap and backlog docs describe *intent*; the **Issues** describe *current work*; **STATUS.md** describes *progress*. When these disagree, Issues + STATUS.md win.

## 3. Repository map

```
.
├── AGENTS.md            ← you are here (AI + contributor guide)
├── CONTRIBUTING.md      human contributor workflow
├── DESIGN.md           design language (draft) + decisions checklist
├── README.md           project overview
├── STATUS.md           AUTO-GENERATED progress (do not hand-edit)
├── .github/workflows/
│   └── update-status.yml   regenerates STATUS.md from issues
└── docs/
    ├── MVP-and-Roadmap.md  the full plan (MVP, hosting, roadmap, risks)
    ├── ARCHITECTURE.md     CloudKit + R2 architecture in brief
    ├── BACKLOG.md          feature backlog (seed for Issues)
    └── brainstorm.md       original idea dump
```
When app code is added (Phase 0), the Xcode project / Swift packages will live under a top-level app folder; update this map and §6 when that happens.

## 4. Architecture in one screen

- **Data** lives in **CloudKit**, in a **CKShare shared zone**. Membership in that zone *is* the group — there is no separate auth allowlist. Sign-in is **Sign in with Apple**.
- **Push** uses **CloudKit subscriptions** (CKQuerySubscription) — fires on record changes, no server.
- **Media** (vlog photos/videos) lives in **Cloudflare R2** (10 GB free, zero egress). Only *metadata + expiry* go in CloudKit.
- **Scheduled jobs** (daily "who's it", expiry, R2 upload signing) run on a **free serverless cron** (Cloudflare Workers / Val Town) — added only when a feature needs it.
- **Sync is near-real-time (seconds), NOT live co-editing.** Don't design features that need Google-Docs-style simultaneous editing without flagging it first.
- Cross-platform fallback if the group ever needs Android: **Firebase (Spark)**. Not active. See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## 5. How work is tracked

Every backlog item is a GitHub Issue with two kinds of labels:

- **Phase** (the *when*): `phase-0: foundations`, `phase-1: mvp`, `phase-2: social`, `phase-3: trivselslekene`, `phase-4: money`, plus `decision` for open product questions.
- **Area** (the *what*): `backend`, `ui`, `ios-native`, `ai`, `infra`.

To find the next thing to do: open Issues, filter by the lowest open phase, pick an unassigned one, assign yourself.

## 6. Build & run

> No app code exists yet. Once Phase 0 lands, document here: Xcode version, how to open the project, signing/CloudKit container setup, and how to run on device / TestFlight. Until then there is nothing to build.

## 7. Contributor workflow

1. **Pick an issue** (lowest open phase first). Assign yourself.
2. **Branch** from `main`: `phase-1/group-chat`, `fix/poll-results`, etc.
3. **Commit** in present tense, reference the issue: `Add poll results view (#11)`.
4. **Open a PR** into `main`, link the issue (`Closes #11`). Keep PRs small.
5. Update any checklist items in the issue as you go.

## 8. Guardrails (do / don't)

**Standing principles — apply to every feature and PR, forever:**

- **Security & privacy by default.** Closed group (CKShare membership), Sign in with Apple, collect the minimum data, encrypted in transit + at rest, server-side permission checks (not just UI). Never weaken these for convenience. No secrets in the repo.
- **No ads, ever.** No ad SDKs, no ad slots, no tracking-for-advertising, no sponsored content. This is a friends-only app and stays ad-free by design.

Plus:

- **Do** keep the app **iOS-only** and distribute **TestFlight-only** for now (avoids App Store user-generated-content review).
- **Do** keep vlog media **ephemeral** (expiring) and stored on **R2**, not CloudKit.
- **Don't** commit secrets: `*.p8`, `*.p12`, `*.mobileprovision`, `GoogleService-Info.plist`, `.env` (already in `.gitignore`).
- **Don't** make the betting / "TL polymarket" feature **hold or route money or take a cut** — Norwegian gambling law. Points-only first; real money only peer-to-peer in Vipps with the app taking nothing. See issue and roadmap §6.
- **Don't** add a feature requiring a paid always-on server — the project is intentionally $0-hosting.
- **Don't** hand-edit `STATUS.md` — it's regenerated.

## 9. Glossary

- **TL / Trivselslederne** — the group / the app ("trivsel" = wellbeing/good vibes).
- **WAD?FC** — the group's name/logo.
- **Trivselslekene** — the yearly games/event the app is built around.
- **Hjemme i Sandnes** — "home in Sandnes"; the calendar feature for marking when you're in town.
- **CKShare / shared zone** — the CloudKit construct that holds the group's shared data.
- **R2** — Cloudflare's object storage (for media).
- **BeReal / daily vlog** — the ephemeral daily clip feature.

## 10. How this file & status stay current

- **STATUS.md is automatic.** The workflow [`.github/workflows/update-status.yml`](.github/workflows/update-status.yml) regenerates it from the Issues on every issue/PR change and once daily. Never edit it by hand.
- **AGENTS.md is deliberately low-maintenance:** it points to live sources instead of duplicating them, so it rarely goes stale. Update it only when something structural changes — the architecture, the repo layout, the labels, or the build/run steps. Treat that update as part of the same PR that made the change.
