# VISION — the rethink: a living space your group shapes

> **Status: north-star (June 2026, idea phase).** The "think differently" document. Argue with it.
>
> ✅ **Decisions now locked ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md)) — these override parts of the text below:** **(1)** v1 customization is **settings/editor-first** (paste scripts/config); the "talk it into being" of Inversion 3 is a **later** bet, v1 only *futureproofs* for AI. **(2)** "One model" (Inversion 2) is **semantically unified but physically separated** — View/Reaction/Theme are independent modules, not one mega-renderer. **(3)** **iOS-only**; **multi-tenant of *private* groups** (any crew creates their own invite-only group; **no public layer** in v1); a person belongs to many groups and **switches like Instagram accounts**.

## 0. The reframe, in one breath

Stop building **a configurable app that an admin sets up**. Build **a living shared space that a group of friends shapes from the inside** — one that works **instantly and offline**, is made of **one simple living model** instead of three bolted-on engines, treats the **AI as a (later) resident member** rather than a feature, and has **ritual, memory, and time** in its bones. Less an app you use; more a **clubhouse you and your friends keep remodelling**. Any crew can spin up their **own private** clubhouse.

This isn't a bigger plan. In several places it's a *smaller, sharper* one — because the right primitives delete whole subsystems.

## 1. What we'd be settling for

Our current architecture is genuinely solid, but be honest about what it is: a competent assembly of patterns Discord/Notion/Salesforce already proved. It would work. It would also feel like "a slightly customizable group chat," carrying an enterprise-shaped spine (admins, permission matrices, cloud-first sync) that a nine-person friend group doesn't need. We do better by inverting five assumptions.

## 2. Five inversions

### Inversion 1 — Local-first, not cloud-first
**Different:** The **group's state lives on each phone** as the source of truth, in a conflict-free log (CRDT) that **merges automatically**. The cloud is *just a transport*. This is the Ink & Switch *local-first* model — own your data, work offline, sync when you can.
**Why it's better:** instant (no spinners), resilient (works with no signal), and the merge log *is* the audit trail. *(Plan B if CRDT proves too hard: CloudKit-as-source-of-truth + optimistic — [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3.)*
**Tamed:** start with simple CRDTs (LWW-register + add-only sets) that cover the MVP; no live co-editing in v1.

### Inversion 2 — One living model, not three engines
**Different:** **One object graph.** Everything is an **object** (Event, Poll, Message, Tradition…); how it's **shown** is a *view*, how it **reacts** is a *reaction*, how it **looks** is a *theme*. Facets over the same objects — like a spreadsheet where cells, formulas, and views are one system.
**Why it's better:** fewer concepts; simpler to build/teach; ideal for a (later) AI.
**Tamed (important):** objects stay **domain-typed** (not generic primitives), *and* **physically separated** in code — View/Reaction/Theme are independent modules, not one mega-renderer ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L2). That's the guard against the Spotify-HubFramework trap ([`PRIOR-ART.md`](PRIOR-ART.md)).

### Inversion 3 — Malleable & AI-native: *later*
> **✅ Reframed ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L1): this is a *later* bet, not v1.** v1 customization is **settings/editor-first** — a creator surface where you pick options, edit tokens, and *paste scripts/config*. The conversation-first vision is the **direction we futureproof for** (validated declarative documents = schema-as-API), but we don't build or depend on it in v1.

**Later:** **Conversation becomes a primary way the space changes**, the AI a **resident of the group**: "make a leaderboard for the ski trip and remind everyone to pack" → the AI proposes objects+views+reactions into the one model; you approve. This is the *malleable software* thesis (Litt et al.) with LLMs as the enabler — emitting **validated data, never code** ([`AI-READINESS.md`](AI-READINESS.md)). Not a v1 bet because on-device-model capability + the UX are unvalidated.

### Inversion 4 — High-trust, small-N — *inside private groups*
> **✅ Decided ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D1): multi-tenant of *private* groups.** Any crew creates their **own invite-only group**, but there is **no public feed/store/strangers** in v1. The high-trust model below is how it works **inside** each private group; there's no public low-trust zone to manage.

**Different:** Inside a group it's **friends who trust each other**, so we **delete most enterprise machinery** — permissions are **light and social**, and the simplicity is reinvested into intimacy, play, and speed.
**Why it's better:** apps built for scale and strangers feel cold and bureaucratic; building *for* high trust and small N is a feeling they structurally can't fake — and a huge amount of code we don't write. Because every group is **private and invite-only**, this holds for *every* tenant.

### Inversion 5 — Ritual, memory, and time as primitives
**Different:** Make **Tradition** (a recurring, optionally-scored ritual with history), a **living timeline/archive** ("this day last year"), and **time** first-class **objects**.
**Why it's better:** it's the soul that keeps the space alive between events, and it's unique to the friend-group domain — a Tradition is just an object with a recurrence + view + reaction + history.
**Tamed:** ship *one* ritual (Trivselslekene or "who's it") first.

## 3. Why this is genuinely better, not just shinier

- **Speed & feel:** local-first = instant, offline — slim/fast/smooth, structurally.
- **Soul:** a clubhouse with rituals + memory beats "configurable group chat."
- **Simplicity:** one model + high-trust private groups = *less* to build (no public-platform machinery, one platform).
- **Durability & ownership:** your history lives on your devices.
- **Portability hygiene:** the thin seam keeps the brain clean (testability + Plan-B swap), even though Android isn't pursued.

## 4. Why it still ships small

1. **Local-first, scoped, is *simpler*** — LWW + add-only sets cover the MVP; CloudKit is still the transport; Plan B exists.
2. **One model = fewer moving parts** than three engines; the walking skeleton ([`DATA-MODEL.md`](DATA-MODEL.md) §10) is create group → invite → one message rendered by one view.
3. **Private + iOS-only + editor-first** removes whole subsystems (public moderation, cross-platform, conversation-first AI) from v1.

## 5. What changes vs. what we keep

**Keep:** data-not-code, App-Store posture, ad-free + privacy-first, performance budgets, the thin seam, App Intents/Siri, on-device AI *as a later goal*.
**Reframe:** SDUI → **views**; rules → **reactions**; AI-readiness → **futureproofing now, the interface later**; one enterprise RBAC → **light permissions inside private groups**.
**New:** local-first CRDT store (with Plan B); one physically-separated object model; **multi-group identity + Instagram-style switching** ([`DATA-MODEL.md`](DATA-MODEL.md) §3); ritual/memory/time as types; settings/editor-first customization.

## 6. What to prototype to believe it (spikes, before any code)

- **Spike L — local-first slice:** CRDT log of messages on two devices; *raised bar:* under concurrent edits + interruptions, does merge match expectations and stay explainable? If not → Plan B.
- **Spike M — one model:** Event + its view + a reaction as *separated* facets; simpler than three engines without going generic?
- **Spike R — a Tradition:** "who's it today" as a single Tradition object (recurrence + view + history).
- *(Spike N — conversation-first AI — deferred per L1.)*

If these feel good, we have something genuinely new. If not, fall back to the conventional plan — nothing lost.

## 7. North star

**Malleable software for your people: a warm, private, living space your group shapes — instant, offline, full of your own rituals and memory. Any crew can make their own. Not an app you configure to death; a place you and your friends keep making your own.**

---

### Sources (verified June 2026)
- [Local-first software — Ink & Switch](https://www.inkandswitch.com/local-first.html) · [Malleable software — Ink & Switch (Litt et al., June 2025)](https://www.inkandswitch.com/essay/malleable-software/) · [Malleable software in the age of LLMs — Geoffrey Litt](https://www.geoffreylitt.com/2023/03/25/llm-end-user-programming.html) · [Automerge](https://automerge.org/)
- Internal: [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md), [`DATA-MODEL.md`](DATA-MODEL.md), [`PRIOR-ART.md`](PRIOR-ART.md), [`AI-READINESS.md`](AI-READINESS.md).
