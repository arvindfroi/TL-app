# AGENTS.md — AI assistant & contributor guide

> **If you are an AI assistant:** read this whole file first, then [`docs/VISION.md`](docs/VISION.md), [`docs/OPEN-DECISIONS.md`](docs/OPEN-DECISIONS.md), and the [`docs/README.md`](docs/README.md) index. For *current* status read the live sources in [Sources of truth](#2-sources-of-truth) — don't rely on memory. Prefer proposing changes as a PR linked to an issue.

This file follows the [AGENTS.md](https://agents.md) convention and is auto-detected by most AI tools. Humans: see also [`CONTRIBUTING.md`](CONTRIBUTING.md).

> ✅ **DIRECTION (June 2026, decided — [`docs/OPEN-DECISIONS.md`](docs/OPEN-DECISIONS.md)).** Idea / pre-development, no app code yet. A **local-first, single-model, malleable "group OS"**: state is **local-first** (cloud = transport); **views/reactions/looks** are facets of *one* domain-typed object model (semantically unified, **physically separated**); customization is **declarative data, never code** and **settings/editor-first** (AI is a *later* connector — v1 only futureproofs). **iOS-only** (Android not pursued). **Multi-tenant of *private* groups**: any crew creates their own invite-only group, **no public feed/store/strangers**; a person belongs to many groups and **switches the active group like Instagram accounts**. Target feel: **Cocoon-Shell UI customization + Minecraft-command-block rules + Snapchat chat, free on CloudKit + R2.** Ad-free, privacy-first. **Only monetization is deferred.** When docs conflict: **[`docs/OPEN-DECISIONS.md`](docs/OPEN-DECISIONS.md) on strategy, else [`docs/VISION.md`](docs/VISION.md) + newest [`docs/DECISIONS.md`](docs/DECISIONS.md) ADR.**

---

## 1. Project snapshot

- **What:** Trivselslederne ("TL") — a **local-first, deeply customizable space for a friend group**: plan hangouts, group chat, and the yearly **Trivselslekene**. Any crew can create their **own private group**; a person can be in **many groups** and switch between them. Each group shapes its own **layouts, design, motion, and automations**; the core jobs ship as a great default. WAD?FC is the first group.
- **Who:** Arvind, Ruben, Fridrik, Morten, Adrian, Fredrik, Emil, Lars, Eivind.
- **Platform:** **iOS-only** (Swift / SwiftUI). Android **not pursued** ([`docs/OPEN-DECISIONS.md`](docs/OPEN-DECISIONS.md) D3); a **thin seam** (core/model import no SwiftUI/CloudKit) is kept only for testability + the CRDT Plan-B swap. Deep iPhone integration (App Intents/Siri, Widgets, on-device Foundation Models).
- **Backend:** **local-first** on-device store as source of truth; **CloudKit** = sync transport + push (one private zone per group); **Cloudflare R2** = media/assets. No server we run. ~$0 hosting; only cost is Apple Developer (~$99/yr).
- **Status:** idea / pre-development. This repo is product + architecture thinking + the issue backlog.
- **Non-negotiables (every phase, every PR):** **privacy by default**, **no ads — ever**, **data-not-code** customization, **~$0 hosting**. See [§8](#8-guardrails-do--dont).

## 2. Sources of truth

| You want to know… | Look here |
|---|---|
| **The full doc map + reading order** | **[`docs/README.md`](docs/README.md)** — the index; start here |
| The vision / current direction | [`docs/VISION.md`](docs/VISION.md) — **north star** |
| **Strategic decisions (locked + deferred)** | [`docs/OPEN-DECISIONS.md`](docs/OPEN-DECISIONS.md) — wins on strategy |
| How it's built | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) (local-first, one model, thin seam) |
| **The data model (foundation)** | [`docs/DATA-MODEL.md`](docs/DATA-MODEL.md) — the one object graph, CRDT/conflict, multi-group identity |
| The make-it-yours experience | [`docs/CREATOR.md`](docs/CREATOR.md) (UI/motion customization, import/export, console) |
| Free-on-CloudKit/R2 proof | [`docs/FEASIBILITY.md`](docs/FEASIBILITY.md) |
| The view / reaction specs | [`docs/SDUI-SPEC.md`](docs/SDUI-SPEC.md) (views) · [`docs/RULES-SPEC.md`](docs/RULES-SPEC.md) (reactions) |
| Real-time chat under local-first | [`docs/REALTIME.md`](docs/REALTIME.md) |
| Stack reasoning (& "why not React/CSS") | [`docs/TECH-STACK.md`](docs/TECH-STACK.md) |
| Slim/fast/smooth budgets | [`docs/PERFORMANCE.md`](docs/PERFORMANCE.md) |
| Stability + smaller binary | [`docs/STABILITY.md`](docs/STABILITY.md) |
| Security / threat model | [`docs/SECURITY.md`](docs/SECURITY.md) |
| AI customization (futureproofing) | [`docs/AI-READINESS.md`](docs/AI-READINESS.md) — AI is a *later* connector |
| Honest risks + how to maximize | [`docs/CRITIQUE.md`](docs/CRITIQUE.md) |
| Systems we learn from | [`docs/PRIOR-ART.md`](docs/PRIOR-ART.md) |
| Why a decision was made | [`docs/DECISIONS.md`](docs/DECISIONS.md) (ADR log) |
| Build plan (spikes, CI, DoR/DoD) | [`docs/EXECUTION-PLAN.md`](docs/EXECUTION-PLAN.md) |
| Backlog / progress | [`docs/BACKLOG.md`](docs/BACKLOG.md) · [`STATUS.md`](STATUS.md) (auto-gen) · Issues |
| Historical context (superseded/deferred) | [`docs/PIVOT.md`](docs/PIVOT.md) · [`docs/MVP-and-Roadmap.md`](docs/MVP-and-Roadmap.md) · [`docs/brainstorm.md`](docs/brainstorm.md) |
| The default theme (type/color/motion) | [`DESIGN.md`](DESIGN.md) (tokens) |

**Rule for AI assistants:** docs describe *intent*; **Issues** + **STATUS.md** describe *current work/progress*. On **strategy**, **[`docs/OPEN-DECISIONS.md`](docs/OPEN-DECISIONS.md) wins**; otherwise **[`docs/VISION.md`](docs/VISION.md) + the newest [`docs/DECISIONS.md`](docs/DECISIONS.md) ADR**; `PIVOT.md`/`MVP-and-Roadmap.md` are **historical**, and their public-platform bits are **deferred**.

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
    ├── OPEN-DECISIONS.md  ⭐ strategic decisions: locked + deferred (wins on strategy)
    ├── ARCHITECTURE.md    ⭐ how it's built (local-first, one model, thin seam)
    ├── DATA-MODEL.md      the one object graph (CRDT/conflict, multi-group identity)
    ├── CREATOR.md         ⭐ the make-it-yours experience (UI/motion, import/export, console)
    ├── FEASIBILITY.md     ⭐ the 3 pillars, free on CloudKit + R2
    ├── DECISIONS.md       architecture decision log (ADRs, newest first)
    ├── CRITIQUE.md        honest risks + how to maximize ("spine vs heart")
    ├── TECH-STACK.md      Swift-only/iOS-only reasoning; "why not React/CSS"
    ├── SDUI-SPEC.md       the VIEW layer (layout documents)
    ├── RULES-SPEC.md      the REACTION layer (the "command blocks")
    ├── REALTIME.md        messaging under local-first
    ├── AI-READINESS.md    AI futureproofing (AI is a later connector)
    ├── PERFORMANCE.md     slim/fast/smooth budgets + playbook — a gate on every PR
    ├── STABILITY.md       crash-resistance + smaller binary
    ├── SECURITY.md        cybersecurity prep (threat model + controls)
    ├── EXECUTION-PLAN.md  spikes, critical path, DoR/DoD, CI
    ├── BACKLOG.md         feature backlog (seed for Issues)
    ├── PRIOR-ART.md       systems we learn from
    ├── PIVOT.md           historical: first pivot (public bits deferred)
    ├── MVP-and-Roadmap.md historical: original plan (scope superseded)
    └── brainstorm.md      original idea dump
```
When app code is added, the Xcode project / Swift packages live under a top-level app folder; update this map and §6 then.

## 4. Architecture in one screen

Full version in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md); the *why* in [`docs/VISION.md`](docs/VISION.md); the **data model** in [`docs/DATA-MODEL.md`](docs/DATA-MODEL.md):

- **Identity:** Sign in with Apple — **one person, many private groups**, switched like Instagram accounts; a `Member` per (person, group).
- **Local-first store (source of truth):** event-sourced / CRDT log per group on each device; instant, offline, conflict-free; snapshots + compaction bound growth. *(Plan B: CloudKit source-of-truth + optimistic — [`docs/OPEN-DECISIONS.md`](docs/OPEN-DECISIONS.md) L3.)*
- **Transport & blobs:** **CloudKit** (one private zone per group, sync + push); **R2** (media/assets, zero egress).
- **One object model:** domain-typed objects with **views/reactions/looks** as facets — *semantically unified, physically separated*.
- **Engines in the binary:** view renderer, reaction evaluator, declarative motion — **compile-once**. **Data, never code** (App Store 2.5.2).
- **Customization is settings/editor-first;** AI is a *later* client of the same validated API.
- **Thin seam:** model/engines/validation are pure Swift behind a few protocols — testability + Plan-B swap, **not** Android.
- **Private multi-tenant:** light permissions inside a group; no public surface in v1.
- **Real-time:** instant local; ~1–2s sync via CloudKit push; ephemeral typing/presence on a lossy lane.

## 5. How work is tracked

Backlog items are GitHub Issues with **phase** + **area** labels. *Issue phases predate the local-first direction and will be re-cut — see [`docs/EXECUTION-PLAN.md`](docs/EXECUTION-PLAN.md) and [`docs/BACKLOG.md`](docs/BACKLOG.md).* To find work: lowest open phase, pick an unassigned issue, assign yourself.

## 6. Build & run

> No app code yet. Once it lands, document Xcode version, opening the project, CloudKit container setup, and running on device / TestFlight.

## 7. Contributor workflow

1. Pick an issue. Assign yourself.
2. Branch from `main`: `feat/local-first-store`, etc.
3. Commit in present tense, reference the issue.
4. Open a small PR into `main`, link the issue (`Closes #N`).
5. Update the issue checklist as you go.

## 8. Guardrails (do / don't)

**Standing principles — every feature and PR, forever:**

- **Privacy by default.** Local-first/own-your-data, Sign in with Apple, minimum data, encrypted in transit + at rest, permission checks in the data layer. No secrets in the repo. (See [`docs/SECURITY.md`](docs/SECURITY.md).)
- **No ads, ever.** No ad SDKs/slots, no ad-tracking, no sponsored content.
- **Data, not code.** All customization — views, reactions, **themes, motion, imported assets** — is declarative data interpreted by engines in the binary (App Store **2.5.2**). Never download/execute code; never the webview "mini-app" route (**4.7**). "Bring your own" = a **declarative file** (Lottie/Rive, JSON), not a script. (React-*like* + CSS-*like*, rendered natively — [`docs/TECH-STACK.md`](docs/TECH-STACK.md).)
- **~$0 hosting.** No paid always-on server. CloudKit + R2 + free serverless cron.

Plus:

- **Do** keep it **iOS-only** ([`docs/OPEN-DECISIONS.md`](docs/OPEN-DECISIONS.md) D3); the **thin seam** (no SwiftUI/CloudKit in core) is for testability + Plan-B, not Android.
- **Do** keep groups **private + invite-only** (D1); **no public feed/store/strangers** — don't build public-platform/UGC machinery in v1.
- **Do** keep customization **settings/editor-first**; the AI assistant is a *later* connector — in v1 only **futureproof** (validated declarative documents). (L1)
- **Do** treat **imported bundles as untrusted** — validate-as-data + sandbox + review-before-run; never reach another group's data or escalate a role; **validate-and-degrade**. ([`docs/SECURITY.md`](docs/SECURITY.md) §4.)
- **Do** enforce **accessibility floors** + **performance budgets** ([`docs/PERFORMANCE.md`](docs/PERFORMANCE.md)) in the engines.
- **Don't** hardcode the nine names (or any single group) — seed WAD?FC via the normal create/invite flow.
- **Don't** make any betting feature **hold/route money or take a cut** (Norwegian gambling law). Points-only first.
- **Don't** commit secrets; **don't** hand-edit `STATUS.md`.

## 9. Glossary

- **TL / Trivselslederne** — the group / the app. **WAD?FC** — the group's name/logo. **Trivselslekene** — the yearly games event. **Hjemme i Sandnes** — the "who's home" calendar.
- **Private group** — an invite-only space (its own CloudKit zone). The app is **multi-tenant of private groups**; a person belongs to many and switches between them ([`docs/DATA-MODEL.md`](docs/DATA-MODEL.md) §3).
- **Local-first** — each device holds the source-of-truth state; cloud is transport ([`docs/REALTIME.md`](docs/REALTIME.md)).
- **CRDT** — conflict-free replicated data type ([`docs/DATA-MODEL.md`](docs/DATA-MODEL.md)).
- **One object model** — domain-typed objects whose **views/reactions/looks** are facets; semantically unified, physically separated.
- **View / Reaction / Look** — a layout / an automation rule / theme tokens + motion.
- **Thin seam** — the cheap discipline: core imports no SwiftUI/CloudKit, talks via protocols ([`docs/OPEN-DECISIONS.md`](docs/OPEN-DECISIONS.md) D3).
- **Bundle** — an exportable/importable design (theme + tokens + motion + assets + layout + rules).
- **App Intents** — exposes actions/content to Siri + Apple Intelligence; also the AI's tool surface.
- **CloudKit / R2** — Apple's sync+push backend / Cloudflare object storage for blobs.

## 10. How this file & status stay current

- **STATUS.md is automatic** ([`.github/workflows/update-status.yml`](.github/workflows/update-status.yml)) — never hand-edit.
- **AGENTS.md points to live sources** instead of duplicating them. Update only when something structural changes, as part of the same PR.
