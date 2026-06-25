# Documentation index

The full doc set, grouped, with a reading order. **New here? Read the four ⭐ docs in order**, then dip into the rest. When docs disagree: on **strategy**, [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) wins; otherwise [`VISION.md`](VISION.md) + the newest [`DECISIONS.md`](DECISIONS.md) ADR.

> **Where we are (June 2026):** idea / pre-development, **no code yet**. A **local-first, single-model, malleable "group OS"** ([`VISION.md`](VISION.md)). **Decided:** **iOS-only**, **multi-tenant of *private* groups** (any crew creates their own invite-only group; no public layer in v1), **Instagram-style group switching**, **settings/editor-first** customization (AI is a later connector). Only **monetization** is deferred — see [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md). Target feel: **Cocoon-Shell UI customization + Minecraft-command-block rules + Snapchat-feel chat, free on CloudKit + R2.**

## ⭐ Start here (reading order)
1. [`VISION.md`](VISION.md) — the north star: what we're really building and why it's different.
2. [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) — the locked strategic calls (and the one deferred).
3. [`ARCHITECTURE.md`](ARCHITECTURE.md) — how it's built in one screen (local-first, one model, thin seam).
4. [`CREATOR.md`](CREATOR.md) — the make-it-yours experience (the heart of the product).

## Direction & decisions
| Doc | What it is |
|---|---|
| [`VISION.md`](VISION.md) | The reframe: local-first, one model, rituals — the five inversions. **Current.** |
| [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) | **Strategic decisions** — locked (iOS-only, private multi-tenant, switching, editor-first, one-model-separated, spikes-first) + deferred (monetization). Wins on strategy. |
| [`DECISIONS.md`](DECISIONS.md) | Architecture decision log (ADRs), newest first. The *why*, fast. |
| [`CRITIQUE.md`](CRITIQUE.md) | Honest red-team: the real risks + how to squeeze the potential. "Build the spine, ship the heart." |

## Architecture & specs
| Doc | What it is |
|---|---|
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | The layered system, the thin seam, free hosting, data flow. **Current.** |
| [`DATA-MODEL.md`](DATA-MODEL.md) | **The foundation:** the one object graph — envelope, IDs/HLC, per-field CRDT + conflict, multi-group identity, op log. Spec this before code. |
| [`TECH-STACK.md`](TECH-STACK.md) | iOS-only reasoning, the thin seam, and "why not React/CSS (and how it's still React/CSS-like)". |
| [`SDUI-SPEC.md`](SDUI-SPEC.md) | The **view** layer: layout documents, component registry, theming, validation/fallback. |
| [`RULES-SPEC.md`](RULES-SPEC.md) | The **reaction** layer: trigger→condition→action DSL (the "command blocks"), sandbox, dedup. |
| [`REALTIME.md`](REALTIME.md) | Messaging under local-first: instant local, near-real-time sync, durable vs ephemeral lanes. |
| [`AI-READINESS.md`](AI-READINESS.md) | Schema-as-API; AI as a *later* client of the validated edit loop; App Intents/Siri. |
| [`PERFORMANCE.md`](PERFORMANCE.md) | Slim/fast/smooth: compile-once, budgets, the fast-&-smooth playbook, a gate on every PR. |
| [`STABILITY.md`](STABILITY.md) | Crash-resistance + a small binary: safe-mode recovery, resource bounds, assets-stream-from-R2. |
| [`SECURITY.md`](SECURITY.md) | Cybersecurity prep: threat model, untrusted-import sandbox, E2EE, privacy, checklist (private-groups scope). |

## Planning
| Doc | What it is |
|---|---|
| [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) | De-risking spikes, critical path, Definition of Ready/Done, CI. |
| [`BACKLOG.md`](BACKLOG.md) | Feature backlog (seed for GitHub Issues). |

## Background & context
| Doc | What it is |
|---|---|
| [`PRIOR-ART.md`](PRIOR-ART.md) | Systems to learn from (Cocoon, Discord, Minecraft, Notion, Airbnb/Spotify SDUI…) + the lessons. |
| [`PIVOT.md`](PIVOT.md) | **Historical** — the first pivot; its **public-platform** framing is **deferred** ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D1). |
| [`MVP-and-Roadmap.md`](MVP-and-Roadmap.md) | **Historical** — original plan. Hosting/risk/vlog detail still useful; roadmap superseded. |
| [`brainstorm.md`](brainstorm.md) | The original idea dump. |

---
*See also the root [`README.md`](../README.md) (project overview) and [`AGENTS.md`](../AGENTS.md). `STATUS.md` is auto-generated — don't hand-edit.*
