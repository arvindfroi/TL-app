# PRIOR-ART — systems to learn from (steal the lessons, skip the scars)

> **Status: research / reference (June 2026, idea phase).** Almost nothing we're building is new in isolation — SDUI, rules engines, multi-tenant customization, plugin sandboxes, and local-first sync are all mature fields. This maps the systems worth studying, the **one thing to steal** from each, and the **scar to avoid**. Companion to [`TECH-STACK.md`](TECH-STACK.md), [`SDUI-SPEC.md`](SDUI-SPEC.md), [`RULES-SPEC.md`](RULES-SPEC.md), [`AI-READINESS.md`](AI-READINESS.md).

## 1. Server-driven UI (our view layer)

| System | Steal this | Scar to avoid |
|---|---|---|
| **Airbnb "Ghost Platform"** | A disciplined SDUI system: server describes screens from a shared schema, native clients render. Invest in tooling and previews. | They had a big team; tooling is most of the cost. |
| **Lyft "Canvas"** | **Protocol Buffers** for the UI schema → compact + **built-in versioning**. Versioning is a first-class concern. | Protobuf is a heavier toolchain than JSON; copy the *versioning rigor*, not necessarily protobuf. |
| **Netflix / App Store / Apple Music** | SDUI works beautifully for **feed/catalog-style** screens that change often. | These are read-heavy lists — the easy case. Forms/interactions are harder. |
| **Spotify "HubFramework" (deprecated)** | The **cautionary tale**: they went *fully generic too early* — a few primitives composable into anything = max flexibility, min readability, and it was killed. | **Our biggest risk.** Antidote: domain-specific components, hybrid native+SDUI, generalize only on real demand. |
| **Microsoft Adaptive Cards** | A **publicly documented JSON UI schema** + renderer per platform + a designer — the closest thing to a textbook for our document format. | Card-shaped; we need richer screens, but the schema discipline transfers. |

**Net:** SDUI pays off only with *appropriate scope + domain components + tooling + versioning*. Keep [`SDUI-SPEC.md`](SDUI-SPEC.md) domain-specific and keep some screens native.

## 2. Rules / automation (our "command blocks")

| System | Steal this | Scar to avoid |
|---|---|---|
| **Apple Shortcuts + App Intents** | The **native** automation vocabulary + a friendly builder; and it's the Siri/Apple-Intelligence surface. Model actions as App Intents → Shortcuts/Siri for free. | Shortcuts' advanced UI gets fiddly — keep our friendly builder simpler. |
| **IFTTT** | The **trigger → action** model + a clean connector catalog; dead-simple for non-technical users. | Too limited for power users — that's why we add an advanced editor. |
| **Zapier / Make** | Multi-step flows, filters, a big **catalog of typed triggers/actions** with clear inputs/outputs. | Complexity + pricing-driven bloat. Keep our action allow-list small and curated. |
| **Home Assistant** | **Declarative automations** + a template community + serious "why did this fire" debugging. Our execution log + dry-run echo this. | Power-user-only ergonomics; the friendly layer is the hard part. |
| **Slack Workflow Builder / Notion & Airtable automations** | How to present "When… If… Then…" to **non-technical** people with pickers, not code. | They hide power; we expose an advanced tier alongside. |
| **Tasker (Android)** | What a true power-user ceiling looks like. | Inscrutable to normal users — two-tier is the answer. |

**Net:** trigger→condition→action with a **typed, closed catalog** + a two-tier (friendly builder + advanced editor) is the proven shape. Borrow Home Assistant's debugging and Shortcuts' App-Intents reuse.

## 3. Multi-tenant customizable communities (our overall shape)

| System | Steal this | Scar to avoid |
|---|---|---|
| **Discord** | **The closest analogue.** Servers = tenants, **roles/permissions** = our RBAC, invites = invites, per-server customization, **bots** = our rules engine, custom emoji/themes = theming. Study roles + invite UX. | Their permission matrix is famously confusing — make ours simpler. |
| **Slack** | Workspaces, an app platform, Workflow Builder; strong invite flows. | Enterprise complexity; not friend-group-shaped. |
| **Notion** | **Blocks + templates + a template gallery** = our customization + sharing, almost one-to-one. The masterclass in approachable "build your own." | Performance suffers as documents grow — heed [`PERFORMANCE.md`](PERFORMANCE.md). |
| **Cocoon Shell** | A shipping **theme builder + live preview + asset packs + a community store + exportable bundles** — near-exact proof of our pillar 1. | Themes a launcher, not a full app; we apply the model to a social app. |
| **Telegram** | Bots + custom themes, lightweight and fast on mobile. | Thin moderation model — not our standard. |

**Net:** Discord is the product whose *structure* we're closest to; Cocoon Shell proves the *customization UX*. When a design question is ambiguous, ask "what does Discord/Cocoon do, and how do we make it simpler for 9 friends?"

## 4. Plugin / extensibility + the safety sandbox

| System | Steal this | Scar to avoid |
|---|---|---|
| **VS Code "contribution points"** | A **declarative manifest** where extensions *declare* what they add and the host renders it — exactly our **capability manifest / registry**. Best-in-class "extend by data, not code." | — |
| **Figma plugins** | A **sandboxed** plugin model with a controlled API surface. | Still had security incidents — sandboxing must be real. |
| **WordPress (themes + plugins)** | The original "customize everything," proof of demand. | **Cautionary:** sprawl → security holes, bloat, breakage. Our fixed registry + closed action allow-list + review-before-run is the deliberate opposite. |
| **Salesforce (Flow, "clicks not code")** | Enterprise gold standard for **metadata-driven, configure-not-code** with governance. | Cautionary on **complexity** — keep ours friend-group-simple. |
| **Roblox / Minecraft** | UGC platforms that solved **trust + moderation at scale** for shared creations — relevant to template-gallery abuse-scanning. | Heavy moderation machinery; we need a lightweight version. |

**Net:** the winning pattern is **declarative manifests + a sandboxed closed capability surface + review before run** — exactly our data-not-code stance. WordPress/Salesforce warn where unbounded customization leads.

## 5. Local-first / sync engines (behind our `SyncEngine` port)

Relevant because CloudKit is our current sync *and* our Apple lock-in ([`TECH-STACK.md`](TECH-STACK.md) §3):

- **Linear's sync engine** — the reference for fast, optimistic, offline-capable app sync; study its model objects + delta sync.
- **Figma's multiplayer** — CRDT-ish real-time for the day we might want live co-editing (currently out of scope).
- **Replicache / ElectricSQL / PowerSync** — local-first sync backends; candidates behind our port if we leave CloudKit.
- **Automerge / Yjs (CRDTs)** — conflict-free merge libraries (don't hand-roll).
- **Bluesky / AT Protocol** — users choosing their own feeds/algorithms; adjacent to our rules engine.

**Net:** keep sync behind a protocol now so any of these is a later adapter, not a rewrite.

## 6. AI surfaces (our down-the-line assistant)

- **Apple App Intents + Apple Intelligence** — the only sanctioned Siri path + the on-device model surface; [`AI-READINESS.md`](AI-READINESS.md) is built on it.
- **Raycast** — AI + an extension/action catalog where the AI invokes *typed commands*, not free code — a clean "AI calls catalog operations" model.
- **Notion AI / Linear's AI** — AI that edits *structured documents* (the same propose→validate→apply loop), not a chatbot bolted on.

**Net:** good AI integrations make the model **operate a typed action catalog over structured data** — why our schema-as-API design is the right foundation.

## 7. The three lessons that matter most for us

1. **Don't over-abstract early (Spotify).** Domain-specific components + hybrid native/SDUI. Generalize only on real demand.
2. **Copy Discord's *shape* and Cocoon's *UX*, simplified.** Tenants, roles, invites, bots, customization — legible for 9 friends, not an enterprise console.
3. **Declarative manifest + closed catalog + review-before-run (VS Code / Adaptive Cards), never WordPress-style open code.** It's the safety model *and* the AI-readiness model in one.

---

### Sources (verified June 2026)
- [A Deep Dive into Airbnb's Server-Driven UI System — Airbnb Engineering](https://medium.com/airbnb-engineering/a-deep-dive-into-airbnbs-server-driven-ui-system-842244c5f5)
- [SDUI: what Airbnb, Netflix, and Lyft learned](https://medium.com/@aubreyhaskett/server-driven-ui-what-airbnb-netflix-and-lyft-learned-building-dynamic-mobile-experiences-20e346265305)
- [Skip — native SwiftUI apps for iOS and Android (GitHub)](https://github.com/skiptools/skip) · [Official Android support in Swift 6.3](https://skip.dev/blog/swift-63-android-support/)
- [Microsoft Adaptive Cards](https://adaptivecards.io/) · [VS Code contribution points](https://code.visualstudio.com/api/references/contribution-points)
- [Cocoon Shell — Theme Builder](https://cocoon-shell.com/wiki/theme-builder/)
- [App Intents — Apple Developer](https://developer.apple.com/documentation/appintents)
