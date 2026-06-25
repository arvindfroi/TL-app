# Documentation index

The full doc set, grouped, with a reading order. **New here? Read the four ⭐ docs in order**, then dip into the rest as needed. When docs disagree, **the current direction wins**: [`VISION.md`](VISION.md) + the newest entries in [`DECISIONS.md`](DECISIONS.md); on strategic forks, [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) wins.

> **Where we are (June 2026):** idea / pre-development, **no code yet**. The plan has evolved from "a customizable multi-tenant app" ([`PIVOT.md`](PIVOT.md)) to a **local-first, single-model, malleable "group OS"** ([`VISION.md`](VISION.md)). Core target: **Cocoon-Shell-style UI customization + Minecraft-command-block rules + Snapchat-feel chat, free on CloudKit + R2.** Key strategic calls are still **open pending Ruben** — see [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md).

## ⭐ Start here (reading order)
1. [`VISION.md`](VISION.md) — the north star: what we're really building and why it's different.
2. [`ARCHITECTURE.md`](ARCHITECTURE.md) — how it's built in one screen (local-first, one object model, ports).
3. [`CREATOR.md`](CREATOR.md) — the make-it-yours experience (the heart of the product).
4. [`FEASIBILITY.md`](FEASIBILITY.md) — the three pillars, and proof they ship **free** on CloudKit + R2.

## Direction & decisions
| Doc | What it is |
|---|---|
| [`VISION.md`](VISION.md) | The reframe: local-first, one model, AI-native, rituals — the five inversions. **Current.** |
| [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) | **The strategic forks** — what's locked (settings-first, one-model-separated, spikes-first) vs open for Ruben (positioning, Android, monetization). Wins on those topics. |
| [`DECISIONS.md`](DECISIONS.md) | Architecture decision log (ADRs), newest first. The *why*, fast. |
| [`CRITIQUE.md`](CRITIQUE.md) | Honest red-team: the real risks + how to squeeze the potential. "Build the spine, ship the heart." |

## Architecture & specs
| Doc | What it is |
|---|---|
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | The layered system, the portable core, free hosting, data flow. **Current.** |
| [`DATA-MODEL.md`](DATA-MODEL.md) | **The foundation:** the one object graph — envelope, IDs/HLC, per-field CRDT + conflict semantics, op log, storage mapping. Spec this before code. |
| [`TECH-STACK.md`](TECH-STACK.md) | Swift-only reasoning, portable-core/native-shell, and "why not React/CSS (and how it's still React/CSS-like)". |
| [`SDUI-SPEC.md`](SDUI-SPEC.md) | The **view** layer: layout documents, component registry, theming, validation/fallback. |
| [`RULES-SPEC.md`](RULES-SPEC.md) | The **reaction** layer: trigger→condition→action DSL (the "command blocks"), sandbox, dedup. |
| [`REALTIME.md`](REALTIME.md) | Messaging under local-first: instant local, near-real-time sync, durable vs ephemeral lanes. |
| [`AI-READINESS.md`](AI-READINESS.md) | Schema-as-API; AI as a *later* client of the validated edit loop; App Intents/Siri; on-device first. |
| [`PERFORMANCE.md`](PERFORMANCE.md) | Slim/fast/smooth: compile-once, budgets, the fast-&-smooth playbook, a gate on every PR. |
| [`STABILITY.md`](STABILITY.md) | Crash-resistance + a small binary: safe-mode recovery, resource bounds, assets-stream-from-R2. |
| [`SECURITY.md`](SECURITY.md) | Cybersecurity prep: threat model, untrusted-import sandbox, E2EE, GDPR/privacy, checklist. |

## Planning
| Doc | What it is |
|---|---|
| [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) | De-risking spikes, critical path, Definition of Ready/Done, CI. |
| [`BACKLOG.md`](BACKLOG.md) | Feature backlog (seed for GitHub Issues). |

## Background & context
| Doc | What it is |
|---|---|
| [`PRIOR-ART.md`](PRIOR-ART.md) | Systems to learn from (Cocoon, Discord, Minecraft, Notion, Airbnb/Spotify SDUI…) + the lessons. |
| [`PIVOT.md`](PIVOT.md) | **Historical** — the first pivot (single-group → multi-tenant). Superseded in scope by `VISION.md`; multi-tenant framing is *proposed* pending [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) O1. |
| [`MVP-and-Roadmap.md`](MVP-and-Roadmap.md) | **Historical** — original plan. Hosting/risk/vlog detail still useful; roadmap superseded by `VISION.md` §8. |
| [`brainstorm.md`](brainstorm.md) | The original idea dump. |

---
*See also the root [`README.md`](../README.md) (project overview) and [`AGENTS.md`](../AGENTS.md) (guide for AI assistants & contributors). `STATUS.md` is auto-generated — don't hand-edit.*
