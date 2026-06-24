# CRITIQUE — limits, honest risks, and how to squeeze the potential

> **Status: red-team + maximize pass (June 2026, idea phase).** A deliberately critical look at everything in [`VISION.md`](VISION.md), [`CREATOR.md`](CREATOR.md), [`REALTIME.md`](REALTIME.md), [`TECH-STACK.md`](TECH-STACK.md). The job: find where this breaks, then turn the constraints into advantages. Ranked by *how much it actually threatens the project*.

## 0. The one-sentence verdict

The architecture is sound and the vision is special; **the real enemy is scope versus a two-person hobby team** — so the way to get the *most* out of this is to build the ambitious *spine* but ship a ruthlessly narrow *heart* first, and let the platform earn its way in.

---

## 1. The threats that could actually sink it (highest severity)

### 1.1 Scope vs. a two-person hobby team — **the #1 risk**
What we've sketched is *five products*: a sync engine, a UI engine, a rules engine, an AI customization layer, and a creator IDE — plus a community gallery. The failure mode isn't a bad design; it's a beautiful architecture that **never ships**.
- **Mitigation:** brutal MVP. The walking skeleton ([`TECH-STACK.md`](TECH-STACK.md) §5) — create group → invite → one fast, local-first, themed chat + one ritual — is the *whole* first release.
- **Buy, don't build, the plumbing:** use a proven CRDT library (Automerge/Yjs-style), Apple's App Intents/Foundation Models, an existing node-editor. **Spend our invention budget only on the genuinely novel parts** (the malleable group experience, rituals).

### 1.2 Local-first/CRDT is the hardest engineering, and the log grows forever
CRDTs are subtle, and an **append-only log grows without bound** — fighting phone storage and the "unlimited time-travel" promise. Schema-evolving a *synced* log across versions is hard.
- **Mitigation:** **snapshots + compaction** (rich recent history + milestone snapshots, not literally-every-keystroke-forever). Start with the **simplest CRDTs** (LWW-registers + add-only sets) that already cover chat/RSVP/polls/points.
- **Reality check:** likely the part that eats months. If it does, fall back to plain CloudKit sync without losing the rest of the vision.

### 1.3 CloudKit is a transport, not a sync engine — and it's a ceiling
Running a CRDT log over CloudKit: record-size/rate limits, ~seconds latency, **no real-time channel** (typing/presence need something else — [`REALTIME.md`](REALTIME.md) §6), **single-owner CKShare zones** (bus-factor), iCloud-only.
- **Mitigation:** keep sync behind the `SyncEngine` port; document the owner/backup story; accept "near-real-time, not sub-second" for v1.
- **Honest limit:** crisp presence and any cross-platform future eventually need a different transport. Design for it; don't pretend CloudKit removes the ceiling.

---

## 2. Threats that constrain the design (medium severity)

### 2.1 The "one unified model" can collapse into an unreadable meta-system
The Spotify HubFramework trap ([`PRIOR-ART.md`](PRIOR-ART.md) §1): elegant on a slide, "everything is a generic node" in practice.
- **Mitigation:** objects stay **domain-typed and opinionated** (`Event`, `Poll`, `Tradition` — not `Node`). Unify the *plumbing*, keep the *types* specific. "Could we make this more generic?" is a smell, not a goal.

### 2.2 The Creator is a desktop-class tool on a phone-sized screen
Figma, n8n, Webflow are big-screen tools for a reason. Edit-in-place and a **node canvas on a phone** are cramped ([`CREATOR.md`](CREATOR.md) §12).
- **Mitigation / reframe:** make **iPad (and maybe Mac) the creator surface**, iPhone the consumer + light-edit surface. "Build it on iPad, live in it on iPhone" is a *strength* (Procreate is iPad-first), not an apology.

### 2.3 "Talk it into being" may overpromise what on-device AI can do
On-device models are small; complex asks are uneven, and AI emits **invalid** configs.
- **Mitigation:** scope AI v1 to **assisted edits + vetted recipes**, not open-ended generation; lean on **validate-and-repair** so failures are graceful, fixable diffs. AI is the *fast on-ramp*; visual/script tiers are the dependable fallback.

### 2.4 The empty-state / paralysis problem
A malleable app can greet you with a blank, confusing nothing. Freedom is a liability at first contact.
- **Mitigation:** ship a **genuinely excellent, opinionated default**. Customization is *discovered*, never required. 90% of groups should be delighted touching nothing.

---

## 3. Threats to the *public platform* ambition (real, worth scoping out)

### 3.1 High-trust/small-N fights the multi-tenant/public goal
We stripped moderation/RBAC because it's 9 trusted friends ([`VISION.md`](VISION.md) Inversion 4). Going **public to "any friend group"** reintroduces strangers, abuse, moderation, support, age-gating — the exact machinery we removed.
- **Recommendation:** treat "**public platform**" as a *later, separate bet*. Nail one magical private group first.

### 3.2 The template/kit store + shared assets
A cross-group store isn't in anyone's private zone — it needs a shared, moderated catalog. **Resolved direction:** make it **third-party** (the user is fine with that); the bundle format is open, so we don't run or moderate a store ([`CREATOR.md`](CREATOR.md) §11). Imported **assets** still carry an IP/licensing caveat — ship a curated open set so most people never import anything dicey.

---

## 4. The quieter risks (watch, don't panic)

- **App Store gray zone:** an over-general rules engine could read as "a platform within the app" (4.7/2.5.2). Keep it declarative, capability-bounded, reviewed.
- **Performance vs. ambition:** every engine fights the [`PERFORMANCE.md`](PERFORMANCE.md) budgets + app size. Discipline is permanent.
- **Device fragmentation:** on-device AI + newest features gate to recent hardware; degrade gracefully.
- **Discoverability of power:** progressive disclosure can hide deep features so well nobody finds them. Gentle nudges ("want to make this yours?").
- **Is deep customization even *wanted*?** Honest possibility: it's a *builder's fantasy* most groups won't use. **Validate early** — if 8 of 9 only ever pick a theme, invest there and keep the deep creator lean for the one tinkerer.

## 5. How to squeeze *all* the potential (turn constraints into moats)

1. **Local-first → ownership + memory as a feature.** The growing log is the group's **living archive** — "this day last year," replay, time-travel, "you own your history." Privacy + offline + permanence is a banner cloud apps can't wave.
2. **AI-native → the moat.** An app you *talk into shape*, privately and on-device, is genuinely novel — as *assisted, recipe-backed* editing.
3. **Small-N high-trust → intimacy as the product.** A warm, fast, simple space *for your people* is a feeling scale-built apps can't fake.
4. **Rituals + memory → the retention engine.** Traditions + history create **switching cost made of feelings** — the deepest moat, cheapest to start (one ritual).
5. **Kits / bundles → a creativity flywheel (later).** Minecraft's superpower is its mods; a safe bundle-sharing system can echo it.
6. **iPad-first Creator → a pro-tool identity.** "Make on iPad, live on iPhone" turns a screen-size limit into a Procreate-like pedigree.
7. **App Intents → free distribution + AI surface.** Siri/Shortcuts/Spotlight for almost no extra work, same catalog the AI uses.

## 6. The sharp recommendation

**Build the spine, ship the heart.**
- **Spine (now, cheap):** portable core + `SyncEngine`/`Store` ports, the single domain-typed model, schema versioning, the capability catalog. Keeps every ambition *possible*.
- **Heart (ship first):** a beautiful default template, fast **local-first chat**, **one ritual**, **theme + motion customization with live preview + the cascade/locking**, and **AI as the friendly on-ramp**. That alone is something no other app gives a friend group.
- **Earn the rest:** node canvas, asset kits, the (third-party) store, public multi-tenant, external AI — each after the heart is loved and demand is proven. If the hard parts (CRDT, AI) stumble, the fallback (plain CloudKit sync, recipe-only AI) still delivers the heart.

**The biggest way to maximize potential is restraint:** a small, soulful, fast, truly-yours group space that a few friends adore beats a sprawling configurable platform that's perpetually 80% built.

---

### Related
[`VISION.md`](VISION.md) · [`CREATOR.md`](CREATOR.md) · [`REALTIME.md`](REALTIME.md) · [`TECH-STACK.md`](TECH-STACK.md) · [`PRIOR-ART.md`](PRIOR-ART.md) · [`PERFORMANCE.md`](PERFORMANCE.md) · [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md).
