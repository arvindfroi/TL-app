# AGENTS.md — AI assistant & contributor guide

> **If you are an AI assistant:** read this whole file first. It tells you what this project is, where the truth lives, how work is tracked, and the rules to follow. When you need *current* status (what's done, what's next), read the live sources listed in [Sources of truth](#sources-of-truth) — do **not** rely on memory or on what this file says about progress. Prefer proposing changes as a pull request that links the relevant issue.

This file follows the [AGENTS.md](https://agents.md) convention and is auto-detected by most AI coding tools. Humans: see also [`CONTRIBUTING.md`](CONTRIBUTING.md).

> ⚠️ **PIVOT IN PROGRESS (June 2026) — read [`docs/PIVOT.md`](docs/PIVOT.md) before acting.** The project pivoted from "one private app for nine friends" to **a multi-tenant, deeply customizable app any friend group can adopt** (admins/invites, user-editable **layouts + design** via an SDUI engine, and a **rules/automation engine**). Still Swift/SwiftUI, iOS-only for now, ad-free, privacy-first. The architecture summary in §4 below has been updated; the two engines have specs in [`docs/SDUI-SPEC.md`](docs/SDUI-SPEC.md) and [`docs/RULES-SPEC.md`](docs/RULES-SPEC.md).

---

## 1. Project snapshot

- **What:** Trivselslederne ("TL") — a **customizable, multi-tenant iPhone app** for organizing friend groups and hosting events. Any group can create a group, **invite** members, assign **admins**, and reshape their app's **layouts, design, and automation rules**. Core jobs (**plan hangouts**, **group chat**, **Trivselslekene**) ship as the default template. WAD?FC is the first group.
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
| Why we pivoted & the new plan | [`docs/PIVOT.md`](docs/PIVOT.md) — **read first** |
| Why the architecture is the way it is | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) (post-pivot, 7 layers) |
| The customization engines | [`docs/SDUI-SPEC.md`](docs/SDUI-SPEC.md) (layouts) · [`docs/RULES-SPEC.md`](docs/RULES-SPEC.md) (automation) |
| How it stays slim/fast/smooth | [`docs/PERFORMANCE.md`](docs/PERFORMANCE.md) — budgets enforced on every PR |
| How AI can help configure/script it | [`docs/AI-READINESS.md`](docs/AI-READINESS.md) — schema-as-API, on-device first, App Intents/Siri, review limits |
| How we build it (readiness, CI, spikes) | [`docs/EXECUTION-PLAN.md`](docs/EXECUTION-PLAN.md) |
| Why a decision was made | [`docs/DECISIONS.md`](docs/DECISIONS.md) (ADR log) |
| The original feature ideas | [`docs/brainstorm.md`](docs/brainstorm.md) |
| The feature backlog (master list) | [`docs/BACKLOG.md`](docs/BACKLOG.md) (mirrored as Issues) |
| The design language (type, color, motion) | [`DESIGN.md`](DESIGN.md) (the default theme; tokens) |
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
    ├── PIVOT.md            ← START HERE: the customization pivot plan (June 2026)
    ├── ARCHITECTURE.md     post-pivot 7-layer multi-tenant architecture
    ├── SDUI-SPEC.md        the customizable layout engine (server-driven UI)
    ├── RULES-SPEC.md       the rules / automation engine ("advanced settings")
    ├── PERFORMANCE.md      slim/fast/smooth budgets — a gate on every PR
    ├── AI-READINESS.md     design so AI can configure/script the app (down the line)
    ├── EXECUTION-PLAN.md   spikes, critical path, Definition of Ready/Done, CI
    ├── DECISIONS.md        architecture decision log (the why)
    ├── MVP-and-Roadmap.md  full plan; scope superseded by PIVOT §8 roadmap
    ├── BACKLOG.md          feature backlog (re-cut for the pivot)
    └── brainstorm.md       original idea dump
```
When app code is added (Phase 0), the Xcode project / Swift packages will live under a top-level app folder; update this map and §6 when that happens.

## 4. Architecture in one screen (post-pivot)

Seven layers — full version in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md):

- **Identity:** **Sign in with Apple** — one person, many groups.
- **Tenancy & access:** **each group is its own CloudKit CKShare zone**; a user participates in every group they're in. The zone boundary is the security wall. (No longer "one zone, no roles.")
- **Roles & permissions:** app-level **RBAC** (`owner`/`admin`/`member`/`invited`) layered over CKShare — governs who edits layouts, writes rules, moderates. Enforced in the data layer, not just the UI.
- **Domain data** in **CloudKit**; **media** (vlogs) on **Cloudflare R2** (metadata + expiry in CloudKit).
- **Customization store:** per-group theme tokens, **layout documents (SDUI)**, and **rule definitions** — versioned, with graceful fallback — in the group zone.
- **Engines in the binary:** the **SDUI renderer** (data → native SwiftUI) and the **rules evaluator** (declarative automation). **Data is downloaded, never code** (App Store **2.5.2**; not the HTML5/JS mini-app path of 4.7).
- **AI assistant (down the line):** another **client of the editor's validated API** — proposes documents the engines re-validate; on-device Foundation Models first, external AI opt-in with consent. See [`docs/AI-READINESS.md`](docs/AI-READINESS.md).
- **Push:** **CloudKit subscriptions** (no server). **Scheduled jobs / scheduled rules:** **free serverless cron** (Workers / Val Town).
- **Sync is near-real-time (seconds), NOT live co-editing** — also true of layout/rule edits (last-write-wins). **Rule execution is deduplicated** so an automation runs once across all devices.
- **Going public adds Guideline 1.2 UGC duties** (EULA, report/block, moderation, age-gating) — see §8 and [`docs/PIVOT.md`](docs/PIVOT.md) §6.
- Cross-platform later: a second **renderer/evaluator over the same documents** via **Skip**, or **Firebase (Spark)**. Not active.

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

Plus (pivot-era):

- **Do** keep customization **data, not code.** Layouts and rules are declarative documents interpreted by engines in the binary (App Store **2.5.2**). Never download/execute code; never take the HTML5/JS mini-app route (Guideline **4.7**).
- **Do** enforce the **zone boundary** and **RBAC** in the data layer. A custom layout or rule must never reach another group's data or escalate privileges. Validate-and-degrade: unknown components/rules fall back, never crash.
- **Do** plan for **public-release UGC duties** (Guideline **1.2 / 1.2.1**): EULA, in-app **report/block**, moderation, **age-gating**. Admins are first-line moderators.
- **Do** keep AI **as a client, not an executor** ([`docs/AI-READINESS.md`](docs/AI-READINESS.md)): AI proposes declarative documents that the engines re-validate (2.5.2); prefer **on-device** Foundation Models; any **external AI** needs Guideline **5.1.2(i)** in-app consent + disclosure and never sees another group's data.
- **Don't** hardcode the nine names (or any single group). Seed WAD?FC through the same create/invite flow every group uses.
- **Do** keep the app **iOS-only** for now, distributing **TestFlight** in early phases; the **public App Store release is Phase 4** (when UGC tooling lands).
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
- **CKShare / shared zone** — the CloudKit construct that holds **one group's** shared data; the zone boundary is the security wall (one zone per group).
- **RBAC** — role-based access control: app-level `owner`/`admin`/`member`/`invited` layered over CKShare.
- **SDUI** — server-driven UI: screens rendered from editable declarative **layout documents** instead of hardcoded Swift. See [`docs/SDUI-SPEC.md`](docs/SDUI-SPEC.md).
- **Rules engine** — the declarative trigger→condition→action automation layer ("advanced settings"). See [`docs/RULES-SPEC.md`](docs/RULES-SPEC.md).
- **Theme tokens** — semantic design variables (color/type/spacing/motion) a group overrides to restyle the whole app.
- **Tenant / group** — one friend group; the app is multi-tenant (many independent groups).
- **Template gallery** — share/import a group's theme + layouts + rules as a starting setup (Phase 4).
- **App Intents** — the framework exposing app actions/content to Siri + Apple Intelligence (the only path as of WWDC 2026); also the AI assistant's internal tool surface. See [`docs/AI-READINESS.md`](docs/AI-READINESS.md).
- **R2** — Cloudflare's object storage (for media).
- **BeReal / daily vlog** — the ephemeral daily clip feature.

## 10. How this file & status stay current

- **STATUS.md is automatic.** The workflow [`.github/workflows/update-status.yml`](.github/workflows/update-status.yml) regenerates it from the Issues on every issue/PR change and once daily. Never edit it by hand.
- **AGENTS.md is deliberately low-maintenance:** it points to live sources instead of duplicating them, so it rarely goes stale. Update it only when something structural changes — the architecture, the repo layout, the labels, or the build/run steps. Treat that update as part of the same PR that made the change.
