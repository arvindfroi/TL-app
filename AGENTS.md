# AGENTS.md — AI assistant & contributor guide

> **If you are an AI assistant:** read this whole file first, then [`docs/VISION.md`](docs/VISION.md) and the [`docs/README.md`](docs/README.md) index. For *current* status (what's done/next) read the live sources in [Sources of truth](#2-sources-of-truth) — don't rely on memory. Prefer proposing changes as a PR linked to an issue.

This file follows the [AGENTS.md](https://agents.md) convention and is auto-detected by most AI tools. Humans: see also [`CONTRIBUTING.md`](CONTRIBUTING.md).

> ⚠️ **CURRENT DIRECTION (June 2026) — read [`docs/VISION.md`](docs/VISION.md) first, then the [`docs/README.md`](docs/README.md) index.** Idea / pre-development, no app code yet. The plan evolved from "a customizable multi-tenant app" ([`docs/PIVOT.md`](docs/PIVOT.md), now historical) to a **local-first, single-model, malleable "group OS"**: the group's state is **local-first** (cloud = transport); **views** (layout), **reactions** (rules), and **looks** (theme/motion) are facets of *one* domain-typed object model; customization is **declarative data, never code**; you reshape it by **talking to it** (on-device AI). Target feel: **Cocoon-Shell UI customization + Minecraft-command-block rules + Snapchat chat, free on CloudKit + R2.** Swift/SwiftUI native, iOS-only for now, ad-free, privacy-first. **When docs conflict, [`docs/VISION.md`](docs/VISION.md) + the newest [`docs/DECISIONS.md`](docs/DECISIONS.md) ADR win.**

---

## 1. Project snapshot

- **What:** Trivselslederne ("TL") — a **local-first, deeply customizable space for a friend group**: plan hangouts, group chat, and the yearly **Trivselslekene**. Any group can shape its own **layouts, design, motion, and automations**; the core jobs ship as a great default. WAD?FC is the first group.
- **Who:** Arvind, Ruben, Fridrik, Morten, Adrian, Fredrik, Emil, Lars, Eivind.
- **Platform:** iOS (Swift / SwiftUI) native, with a **portable core** behind protocols so Android (Skip) stays possible. Deep iPhone integration (App Intents/Siri, Widgets, Live Activities, on-device Foundation Models).
- **Backend:** **local-first** on-device store as source of truth; **CloudKit** = sync transport + push; **Cloudflare R2** = media/assets. No server we run. ~$0 hosting; only cost is Apple Developer (~$99/yr).
- **Status:** idea / pre-development. This repo is product + architecture thinking + the issue backlog.
- **Non-negotiables (every phase, every PR):** **privacy by default**, **no ads — ever**, **data-not-code** customization, **~$0 hosting**. See [§8](#8-guardrails-do--dont).

## 2. Sources of truth

| You want to know… | Look here |
|---|---|
| **The full doc map + reading order** | **[`docs/README.md`](docs/README.md)** — the index; start here |
| The vision / current direction | [`docs/VISION.md`](docs/VISION.md) — **north star** |
| How it's built | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) (local-first, one model, ports) |
| The make-it-yours experience | [`docs/CREATOR.md`](docs/CREATOR.md) (UI/motion customization, import/export, console) |
| Free-on-CloudKit/R2 proof | [`docs/FEASIBILITY.md`](docs/FEASIBILITY.md) |
| The view / reaction specs | [`docs/SDUI-SPEC.md`](docs/SDUI-SPEC.md) (views) · [`docs/RULES-SPEC.md`](docs/RULES-SPEC.md) (reactions) |
| Real-time chat under local-first | [`docs/REALTIME.md`](docs/REALTIME.md) |
| Stack reasoning (& "why not React/CSS") | [`docs/TECH-STACK.md`](docs/TECH-STACK.md) |
| Slim/fast/smooth budgets | [`docs/PERFORMANCE.md`](docs/PERFORMANCE.md) |
| AI customization (schema-as-API) | [`docs/AI-READINESS.md`](docs/AI-READINESS.md) |
| Honest risks + how to maximize | [`docs/CRITIQUE.md`](docs/CRITIQUE.md) |
| Systems we learn from | [`docs/PRIOR-ART.md`](docs/PRIOR-ART.md) |
| Why a decision was made | [`docs/DECISIONS.md`](docs/DECISIONS.md) (ADR log) |
| Build plan (spikes, CI, DoR/DoD) | [`docs/EXECUTION-PLAN.md`](docs/EXECUTION-PLAN.md) |
| Backlog / progress | [`docs/BACKLOG.md`](docs/BACKLOG.md) · [`STATUS.md`](STATUS.md) (auto-gen) · Issues |
| Historical context (superseded scope) | [`docs/PIVOT.md`](docs/PIVOT.md) · [`docs/MVP-and-Roadmap.md`](docs/MVP-and-Roadmap.md) · [`docs/brainstorm.md`](docs/brainstorm.md) |
| The default theme (type/color/motion) | [`DESIGN.md`](DESIGN.md) (tokens) |

**Rule for AI assistants:** docs describe *intent*; **Issues** + **STATUS.md** describe *current work/progress*. When vision/architecture docs disagree, **[`docs/VISION.md`](docs/VISION.md) + the newest [`docs/DECISIONS.md`](docs/DECISIONS.md) ADR win**; treat `PIVOT.md` and `MVP-and-Roadmap.md` as **historical**.

## 3. Repository map

```
.
├── AGENTS.md            ← you are here (AI + contributor guide)
├── CONTRIBUTING.md      human contributor workflow
├── DESIGN.md            the default theme (tokens) + checklist
├── README.md            project overview
├── STATUS.md            AUTO-GENERATED progress (do not hand-edit)
├── .github/workflows/
│   └── update-status.yml   regenerates STATUS.md from issues
└── docs/
    ├── README.md          ← the documentation index + reading order
    ├── VISION.md          ⭐ north star: local-first, one model, malleable group OS
    ├── ARCHITECTURE.md    ⭐ how it's built (local-first, one model, ports)
    ├── CREATOR.md         ⭐ the make-it-yours experience (UI/motion, import/export, console)
    ├── FEASIBILITY.md     ⭐ the 3 pillars, free on CloudKit + R2
    ├── DECISIONS.md       architecture decision log (ADRs, newest first)
    ├── CRITIQUE.md        honest risks + how to maximize ("spine vs heart")
    ├── TECH-STACK.md      Swift-only reasoning; "why not React/CSS"
    ├── SDUI-SPEC.md       the VIEW layer (layout documents)
    ├── RULES-SPEC.md      the REACTION layer (the "command blocks")
    ├── REALTIME.md        messaging under local-first
    ├── AI-READINESS.md    AI as a client of the validated edit loop
    ├── PERFORMANCE.md     slim/fast/smooth budgets — a gate on every PR
    ├── EXECUTION-PLAN.md  spikes, critical path, DoR/DoD, CI
    ├── BACKLOG.md         feature backlog (seed for Issues)
    ├── PRIOR-ART.md       systems we learn from
    ├── PIVOT.md           historical: first pivot (single-group → multi-tenant)
    ├── MVP-and-Roadmap.md historical: original plan (scope superseded)
    └── brainstorm.md      original idea dump
```
When app code is added, the Xcode project / Swift packages live under a top-level app folder; update this map and §6 then.

## 4. Architecture in one screen

Full version in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md); the *why* in [`docs/VISION.md`](docs/VISION.md):

- **Identity:** Sign in with Apple — one person, many groups.
- **Local-first store (source of truth):** an event-sourced / CRDT log on each device; instant, offline, conflict-free merge; snapshots + compaction bound its growth.
- **Transport & blobs (not the brain):** **CloudKit** moves changes + push; **Cloudflare R2** holds media/assets (zero egress).
- **One object model:** domain-typed objects (`Event`, `Poll`, `Message`, `Tradition`…) with **views** (layout), **reactions** (rules), and **looks** (theme/motion) as facets — not three engines, not generic primitives.
- **Engines in the binary:** view renderer (data→native SwiftUI), reaction evaluator, declarative motion — **compile-once** for speed. **Data is downloaded, never code** (App Store 2.5.2; not the 4.7 webview path).
- **Creator + AI:** the [`docs/CREATOR.md`](docs/CREATOR.md) and the on-device AI are both clients of one validated **propose→preview→approve** edit API.
- **Portable core:** model/engines/validation are pure Swift behind ports (`Store`, `SyncEngine`, `BlobStore`, `Push`, `AIProvider`) with Apple adapters — so Android/backend-swap is an adapter, not a rewrite.
- **Two trust zones:** warm + light inside a group; validate-and-review at the edges (imports, any public sharing).
- **Real-time:** instant local; near-real-time sync (~1–2s) via CloudKit push; ephemeral signals (typing/presence) on a separate lossy lane.

## 5. How work is tracked

Backlog items are GitHub Issues with **phase** labels (`phase-0`…`phase-4`, plus `decision`) and **area** labels (`backend`, `ui`, `ios-native`, `ai`, `infra`). *Note: issue phases predate the local-first direction and will be re-cut — see [`docs/EXECUTION-PLAN.md`](docs/EXECUTION-PLAN.md) and [`docs/BACKLOG.md`](docs/BACKLOG.md).* To find work: lowest open phase, pick an unassigned issue, assign yourself.

## 6. Build & run

> No app code yet. Once it lands, document Xcode version, opening the project, CloudKit container setup, and running on device / TestFlight. Until then there's nothing to build.

## 7. Contributor workflow

1. Pick an issue (lowest open phase). Assign yourself.
2. Branch from `main`: `feat/local-first-store`, `fix/poll-results`, etc.
3. Commit in present tense, reference the issue: `Add poll results view (#11)`.
4. Open a small PR into `main`, link the issue (`Closes #11`).
5. Update the issue checklist as you go.

## 8. Guardrails (do / don't)

**Standing principles — every feature and PR, forever:**

- **Privacy by default.** Local-first/own-your-data, Sign in with Apple, minimum data, encrypted in transit + at rest, permission checks in the data layer (not just UI). No secrets in the repo.
- **No ads, ever.** No ad SDKs/slots, no ad-tracking, no sponsored content.
- **Data, not code.** All customization — views, reactions, **themes, motion/animation, imported assets** — is declarative data interpreted by engines in the binary (App Store **2.5.2**). Never download/execute code; never the HTML5/JS webview "mini-app" route (**4.7**). "Bring your own" = bring a **declarative file** (Lottie/Rive, JSON), not a script. (It's React-*like* + CSS-*like*, rendered natively — see [`docs/TECH-STACK.md`](docs/TECH-STACK.md).)
- **~$0 hosting.** No feature that needs a paid always-on server. CloudKit + R2 + free serverless cron.

Plus:

- **Do** keep the **portable core** clean: model/engines/validation import no SwiftUI and no CloudKit; Apple specifics live behind ports/adapters.
- **Do** respect the **two trust zones**: light inside a group; validate-as-data + review-before-run for imported bundles/rules; a custom view/reaction can never reach another group's data or escalate a role; **validate-and-degrade** (unknowns fall back, never crash).
- **Do** keep AI **a client, not an executor** ([`docs/AI-READINESS.md`](docs/AI-READINESS.md)): it proposes documents the engines re-validate; **on-device first**; any **external AI** needs Guideline **5.1.2(i)** in-app consent + disclosure and never sees another group's data.
- **Do** enforce **accessibility floors** (Dynamic Type, contrast, Reduce Motion) and **performance budgets** ([`docs/PERFORMANCE.md`](docs/PERFORMANCE.md)) in the engines — a custom theme/motion can't make the app unusable or janky.
- **Don't** hardcode the nine names (or any single group) — seed WAD?FC via the normal create/invite flow.
- **Don't** make any betting feature **hold/route money or take a cut** (Norwegian gambling law). Points-only first.
- **Don't** commit secrets (`*.p8`, `*.p12`, `*.mobileprovision`, `.env` — in `.gitignore`); **don't** hand-edit `STATUS.md`.

## 9. Glossary

- **TL / Trivselslederne** — the group / the app ("trivsel" = wellbeing/good vibes). **WAD?FC** — the group's name/logo. **Trivselslekene** — the yearly games event. **Hjemme i Sandnes** — the "who's home" calendar.
- **Local-first** — each device holds the source-of-truth state; cloud is transport; instant + offline. See [`docs/REALTIME.md`](docs/REALTIME.md).
- **CRDT** — conflict-free replicated data type: edits from many devices merge automatically.
- **One object model** — domain-typed objects whose **views/reactions/looks** are facets ([`docs/VISION.md`](docs/VISION.md)).
- **View** — a layout (the SDUI layer, [`docs/SDUI-SPEC.md`](docs/SDUI-SPEC.md)). **Reaction** — an automation rule (the "command blocks", [`docs/RULES-SPEC.md`](docs/RULES-SPEC.md)). **Look** — theme tokens + declarative motion.
- **Token / cascade** — semantic design variables that resolve Device → Personal → Group-shared → Default ([`docs/CREATOR.md`](docs/CREATOR.md) §8).
- **Bundle** — an exportable/importable design (theme + tokens + motion + assets + layout + rules) as a manifest + assets.
- **Portable core / ports & adapters** — framework-agnostic Swift brain behind protocols ([`docs/TECH-STACK.md`](docs/TECH-STACK.md)).
- **App Intents** — the framework exposing actions/content to Siri + Apple Intelligence; also the AI's tool surface.
- **CloudKit / R2** — Apple's sync+push backend / Cloudflare object storage for blobs.

## 10. How this file & status stay current

- **STATUS.md is automatic** ([`.github/workflows/update-status.yml`](.github/workflows/update-status.yml)) — never hand-edit.
- **AGENTS.md points to live sources** instead of duplicating them. Update it only when something structural changes (architecture, repo layout, labels, build/run), as part of the same PR.
