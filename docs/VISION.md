# VISION — the rethink: a living space your group shapes by talking to it

> **Status: provocation / north-star (June 2026, idea phase).** This is the "think differently" document. The plan we have ([`PIVOT.md`](PIVOT.md) → engines, RBAC, gallery) is a *good configurable app*. This argues for something with more soul and a better spine — and shows it can still ship small. Argue with it.
>
> 🔧 **Read [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) alongside this.** Three refinements now override parts of the text below: **(1)** v1 customization is **settings/editor-first** (paste scripts/config); the "talk it into being" of Inversion 3 is a **later** bet, and v1 only *futureproofs* for AI. **(2)** "One model" (Inversion 2) is **semantically unified but physically separated** — View/Reaction/Theme are independent modules, not one mega-renderer. **(3)** niche-vs-mass positioning (Inversion 4) is an **open decision pending Ruben**, leaning us-first.

## 0. The reframe, in one breath

Stop building **a configurable app that an admin sets up**. Build **a living shared space that a group of friends shapes from the inside by talking to it** — one that works **instantly and offline**, is made of **one simple living model** instead of three bolted-on engines, treats the **AI as a resident member** rather than a feature, and has **ritual, memory, and time** in its bones. Less an app you use; more a **clubhouse you and your friends keep remodelling**.

This isn't a bigger plan. In several places it's a *smaller, sharper* one — because the right primitives delete whole subsystems.

## 1. What we'd be settling for

Our current architecture is genuinely solid, but be honest about what it is: **SDUI engine + rules engine + RBAC + CloudKit sync + AI-as-client.** That's a competent assembly of patterns Discord/Notion/Salesforce already proved. It would work. It would also feel like "a slightly customizable group chat," and it carries an enterprise-shaped spine (admins, permission matrices, cloud-first sync) that a nine-person friend group doesn't need. We can do better by inverting five assumptions.

## 2. Five inversions

### Inversion 1 — Local-first, not cloud-first
**Conventional (our last plan):** CloudKit is the source of truth; the app reads/writes the cloud; "near-real-time, seconds." Offline is an edge case.
**Different:** The **group's state lives on each phone** as the source of truth, in a conflict-free log (CRDT) that **merges automatically**. The cloud (CloudKit today, anything tomorrow) is *just a transport* that ships changes between devices. This is the Ink & Switch *local-first* model — own your data, work offline, sync when you can.
**Why it's better:** It directly delivers our **slim/fast/smooth** goal *at the architecture level* — every action is instant because it's local; there are no spinners and no "waiting on the network." It's resilient (works on a mountain with no signal — relevant for a Norwegian friend group). It **breaks the CloudKit lock-in** ([`TECH-STACK.md`](TECH-STACK.md) §3): sync is a swappable port, so Android/portability stop being a rewrite. And the merge log *is* the audit trail the AI reads.
**The risk & how we tame it:** CRDTs add merge complexity. We tame it by starting with simple, well-understood structures (last-writer-wins registers + add-only sets cover chat/RSVP/polls/points) and only reaching for richer CRDTs if a feature truly needs them. We do **not** chase live co-editing in v1. *(And we name a Plan B: if CRDT-over-CloudKit proves uncontrollable, fall back to CloudKit-as-source-of-truth + optimistic updates — [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3, [`DATA-MODEL.md`](DATA-MODEL.md).)*

### Inversion 2 — One living model, not three engines
**Conventional:** three parallel JSON dialects — layout documents (SDUI), rule documents, theme tokens — plus a separate data model. Four things to learn, four things for AI to juggle.
**Different:** **One object graph.** Everything in a group is an **object** (an Event, Poll, Message, Member, Tradition, Points-tally…). How objects are shown is a **view** (that's "layout"). How objects react to change is a **reaction** (that's "rules"). How they look is a **theme**. Views, reactions, and themes aren't separate engines — they're **facets over the same reactive objects**, like a spreadsheet where cells, formulas, and views are one system. This is the proven "media editor with optional programmability" lineage: spreadsheets, HyperCard, Notion, Airtable.
**Why it's better:** Fewer concepts → simpler to build, simpler to teach, dramatically simpler for the **AI** (one schema, not three). Composability: a "Tradition" object can carry its own view + reaction + points in one place. It collapses our doc set's three engines into one coherent core.
**The risk & how we tame it (important):** "One generic model" is *exactly* the trap that killed Spotify's HubFramework ([`PRIOR-ART.md`](PRIOR-ART.md) §1) — going fully generic too early. We avoid it by making the objects **domain-typed and opinionated** (a real `Event`, a real `Poll`, a real `Tradition` with meaning and good defaults), **not** a soup of nameless primitives. *And by **physical separation** ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L2): the View/Reaction/Theme implementations are independent modules behind interfaces — unified in concept and plumbing, not in one mega-renderer.* Unification is at the *semantic* layer; the components stay specific and readable.

### Inversion 3 — Malleable & AI-native: you talk the app into being
> **⚠️ Reframed (June 2026, [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L1): this is a *later* bet, not v1.** v1 customization is **settings/editor-first** — a creator surface where you pick options, edit tokens, and *paste scripts/config*. The conversation-first vision below is the **direction we futureproof for** (validated declarative documents = schema-as-API), but we don't build or depend on it in v1.

**Conventional:** AI is a "client of the editor" you add later. Customization is done by hand in an editor; AI assists.
**Different (later):** **Conversation becomes a primary way the space changes**, and the AI is a **resident of the group**, not a tool. "Make a leaderboard for the ski trip and remind everyone to pack the night before" → the AI proposes objects+views+reactions into the one model; you glance, approve, done. This is the June 2025 *malleable software* thesis (Litt et al.): users reshape tools at runtime instead of waiting for a vendor — and **LLMs are the missing enabler**.
**Why it's better (later):** It's the genuinely novel product stance — **an app that becomes what each group needs by being asked**, privately and on-device (Apple Foundation Models). It's *more* App-Store-safe, not less: the AI emits **validated data, never code** ([`AI-READINESS.md`](AI-READINESS.md)).
**The risk & how we tame it:** On-device models are small and the UX is unvalidated — which is exactly why it's **not a v1 bet**. We tame it with the propose→preview→approve loop (a wrong proposal is a rejected diff, never a broken app), vetted "recipes," and opt-in external models only for heavy lifts — *when we get there*.

### Inversion 4 — High-trust and small-N *by design*
> **⚠️ Open decision ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) O1):** niche (us-first) vs mass (multi-tenant public) is **not yet locked** — pending Ruben, leaning us-first. The text below is the us-first case.

**Conventional:** RBAC with owner/admin/member/invited, moderation pipelines, anti-abuse — the machinery every public platform needs.
**Different:** Inside a group it's **~9 friends who trust each other**, so we **delete most of that machinery** and treat trust as the default. Permissions become **light and social** ("everyone can do almost everything; a couple of soft guardrails and an undo"), not an enterprise matrix. The simplicity we save gets **reinvested into intimacy, play, and speed**.
**Why it's better:** Most group apps are built for scale and strangers and feel cold and bureaucratic as a result. Building *for* high trust and small N is a feature competitors structurally can't copy — and it's a huge amount of code we simply don't write.
**The risk & how we tame it:** *If* we ever open to other groups (O1), the public edges need safety — a separate **"low-trust zone"** with moderation/consent ([`PIVOT.md`](PIVOT.md) §6). While us-first, that complexity simply doesn't exist.

### Inversion 5 — Ritual, memory, and time as primitives
**Conventional:** Trivselslekene, "who's it," recurring hangouts, the photo history — all built as one-off features.
**Different:** Friend groups *run on* repeated rituals and accumulated memory. So make **Tradition** (a recurring, optionally-scored ritual with history), a **living timeline/archive** ("this day last year," the running story of the group), and **time itself** first-class **objects** in the model — not features bolted on.
**Why it's better:** It's what makes the space *theirs* and keeps it alive between events; it's the soul that turns a utility into a place you want to open. And it's unique to the friend-group domain — it falls out naturally from Inversion 2's object model (a Tradition is just an object with a recurrence + a view + a reaction + a history).
**The risk:** Scope. We ship *one* ritual (Trivselslekene or "who's it") as the first Tradition and let the primitive prove itself before generalizing.

## 3. Why this is genuinely better, not just shinier

- **Speed & feel:** local-first = instant, offline, no spinners — the slim/fast/smooth goal, achieved structurally rather than fought for.
- **Soul:** a clubhouse with rituals and memory beats "configurable group chat." This is the part people will love.
- **Simplicity:** one model + high-trust defaults = *less* to build than three engines + full RBAC. Bold vision, smaller core.
- **Durability & ownership:** your group's history lives on your devices, not hostage to one cloud account — and survives backend changes.
- **AI that fits (later):** one schema + a merge-log audit trail + a propose/approve loop is the ideal substrate for a private, on-device assistant.
- **Portability:** sync-as-transport + a thin seam means Android/longevity are adapters, not rewrites (O2).

## 4. The honest risks (and why it still ships small)

The danger with a vision like this is over-engineering and never shipping. Three guards:

1. **Local-first makes the core *simpler*, not harder, if scoped.** LWW-register + add-only-set CRDTs cover the whole MVP; CloudKit is still the transport we already planned; and there's a Plan B if merge gets hairy. We are not building Figma multiplayer.
2. **One model means *fewer* moving parts than the three-engine plan**, so the walking skeleton ([`DATA-MODEL.md`](DATA-MODEL.md) §10) gets *easier*: create group → invite → one object (a message) rendered by one view, synced by the local-first log. Everything else is new object *types*, not new engines.
3. **Everything here maps onto the existing plan**, so it's a sharpening, not a restart. We keep the App-Store-safe, ad-free, privacy-first, data-not-code commitments unchanged.

## 5. What changes vs. what we keep

**Keep (most of it):** data-not-code, App-Store posture (2.5.2/4.7/1.2/5.1.2(i)), ad-free + privacy-first, the performance budgets, the thin-seam/native-shell split, App Intents/Siri, on-device AI *as a later goal*.

**Reframe:**
- SDUI "layout documents" → **views** (a facet of the one model). [`SDUI-SPEC.md`](SDUI-SPEC.md) mostly survives as the view layer.
- Rules "documents" → **reactions** (another facet). [`RULES-SPEC.md`](RULES-SPEC.md) survives as reaction semantics.
- AI-readiness → **futureproofing now, the interface later** ([`AI-READINESS.md`](AI-READINESS.md), [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L1).
- RBAC → light inside a group; the multi-tenant question is open (O1).

**New:**
- A **local-first, event-sourced, CRDT** store with sync-as-transport (the biggest change; revises [`ARCHITECTURE.md`](ARCHITECTURE.md) layers 2–4), with a documented Plan B.
- A **single unified object model** (collapses three engines), physically separated in code.
- **Ritual / memory / time** as first-class object types.
- **Settings/editor-first** customization; conversation-first as a later exploration.

## 6. What to prototype to believe it (cheap spikes, before any commitment)

- **Spike L — local-first slice:** a CRDT log of "messages" on two devices, merging offline edits, CloudKit as transport. *Raised bar:* under concurrent edits + interruptions + multi-device, does merge match expectations, and can we explain "why was my edit overwritten?" If not → Plan B.
- **Spike M — one model:** model an Event + its view + a reaction as separated facets of one object; render natively. Simpler than three engines, *without* going generic?
- **Spike N — (later) talk it into being:** on-device Foundation Models turning a request into validated objects/views/reactions, shown as an approve-able diff. *Deferred per L1.*
- **Spike R — a Tradition:** implement "who's it today" or Trivselslekene as a single Tradition object with recurrence + view + history. Does the primitive generalize?

If these feel good, we have something genuinely new. If they don't, we fall back to the (already solid) conventional plan — nothing lost.

## 7. North star

**Malleable software for your people: a warm, living space your group shapes — instant, offline, private, and full of your own rituals and memory. Not an app you configure to death; a place you and your friends keep making your own.**

---

### Sources (verified June 2026)
- [Local-first software: you own your data, in spite of the cloud — Ink & Switch](https://www.inkandswitch.com/local-first.html)
- [Malleable software: restoring user agency in a world of locked-down apps — Ink & Switch (Litt et al., June 2025)](https://www.inkandswitch.com/essay/malleable-software/)
- [Malleable software in the age of LLMs — Geoffrey Litt](https://www.geoffreylitt.com/2023/03/25/llm-end-user-programming.html)
- [Automerge — a CRDT for local-first software](https://automerge.org/)
- Internal: [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md), [`PRIOR-ART.md`](PRIOR-ART.md), [`TECH-STACK.md`](TECH-STACK.md), [`AI-READINESS.md`](AI-READINESS.md).
