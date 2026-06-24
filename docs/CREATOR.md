# CREATOR — making it *fun* to build your own experience

> **Status: design (June 2026, idea phase).** The make-or-break problem behind [`VISION.md`](VISION.md): when a group can change **fonts, colors, imported graphics, components, menu icons, motion, rules, and policies**, that's hundreds of parameters. If creating feels like an enterprise admin console, the whole idea dies. This designs the **Creator** so it feels like Procreate or Minecraft creative mode — powerful, playful, hard to break. Companion to [`SDUI-SPEC.md`](SDUI-SPEC.md), [`RULES-SPEC.md`](RULES-SPEC.md), [`AI-READINESS.md`](AI-READINESS.md).

## 1. The real problem: parameter explosion

Total freedom = total overwhelm. A blank screen with 400 knobs is the worst possible UX. Every system that nailed deep customization solved this the same way: **most people touch very little, power users can touch everything, and nobody sees all of it at once.** The design below is five principles that make that true, then the surfaces that deliver it.

## 2. Five principles that tame the chaos

1. **Sparse overrides over a complete default.** There's always a full, beautiful **default experience** (the shipped template). A group's customization is only the *diff* from it — you change the 3 things you care about, the other 397 inherit. You are never confronted with "all params"; you're confronted with *the one you're changing*.
2. **Tokens + cascade, not raw values.** You change a **semantic token** ("accent color", "corner roundness", "heading font") once and it ripples everywhere, instead of editing 100 components. Layered inheritance (app → screen → component) means you edit at the altitude you care about.
3. **Progressive disclosure — doors, not dashboards.** Four depth levels, each a door to the next, so beginners never see the deep end: **Pick → Arrange → Wire → Script** (§3). 90% of people stop at Pick/Arrange.
4. **Safe by construction.** Accessibility floors and performance budgets are **guardrails the Creator enforces live** — you literally can't set unreadable contrast or a janky layout without a visible warning. Freedom inside rails you can't fall off.
5. **Always-live, always-reversible.** Every change previews **instantly on the real app** (not a mockup), and the CRDT history ([`REALTIME.md`](REALTIME.md)) gives **unlimited undo / time-travel**. Experimentation is free because nothing is permanent.

## 3. The four creation depths (progressive disclosure)

| Depth | Who | What it feels like | Analogy |
|---|---|---|---|
| **1. Pick** | everyone | Choose a template, swap a theme, drop in an imported font/brush/icon pack, flip toggles. No building. | Procreate: choosing a brush set |
| **2. Arrange** | curious members | **Visual builder, live**: enter "edit mode" on the real screen, drag sections, add/remove components, tweak tokens with sliders/swatches and watch it change. | iOS home-screen jiggle mode / Webflow-lite |
| **3. Wire** | tinkerers | **Node canvas** (n8n-style): connect triggers → conditions → actions to automate the group. | n8n / Unreal Blueprints / MC command blocks |
| **4. Script** | power users | **Raw params + the rule/document text**: the command-palette and the underlying DSL for total control. | Minecraft command blocks / config files |

Crucially, **3 and 4 are the same thing in two views** (§5) — and **1–2 produce the same documents** as 3–4, so there's no "second-class" tier. You can start by talking to the AI ([`AI-READINESS.md`](AI-READINESS.md)), then open any depth to refine.

## 4. The Creator's surfaces (one "Workshop", four tabs)

A single **Creator / Build mode** ("the Workshop"), entered from the real app, with four surfaces over the **one shared model** ([`VISION.md`](VISION.md) Inversion 2):

- **Design** — the visual builder + token editor. Edit-in-place on a live preview; pick components from a palette; adjust tokens (color/type/spacing/radius/motion) with immediate feedback; manage component **variants**.
- **Logic** — the **node canvas**: triggers, conditions, and actions as wired blocks; plus a **"Script" toggle** showing the same logic as the rule document. Edit either; they stay in sync.
- **Assets** — the **bring-your-own library** (§6): import fonts, images, icon packs, sounds; organize; package as **kits**.
- **Console** — the **debugging + observability toolkit** (§7): live event log, inspector, "why did this happen" trace, simulate/dry-run, validation warnings, preview-as-role/as-device.

A persistent **Preview** runs alongside (or you edit directly on the live screen), with a **draft → publish** control (§8) and an **undo/history** scrubber.

## 5. Logic: node canvas ↔ script (the n8n + command-block ask)

The automation surface is **dual-representation**, because tinkerers and power users want different doors to the same thing:

- **Node canvas:** drag a **trigger** ("event created"), wire it into **conditions** ("on a weekend"), into **actions** ("create a poll", "notify everyone"). Visual, legible, no syntax. Great for "get the idea as you go."
- **Script view:** the exact same automation as the declarative rule document ([`RULES-SPEC.md`](RULES-SPEC.md)) — editable as text, with autocomplete from the **capability catalog**. Great for speed and precision.
- They are **two renderings of one underlying document**; flipping views never loses work. (This is the Blueprints↔C++, Scratch↔text, Shortcuts↔text pattern.)
- A **command palette** ("/remind everyone 1 day before any event") is the fastest path for power users — the Minecraft command-block feel — and it just emits the same document.

Because it's all the rule **document** underneath, it stays **data-not-code / App-Store-safe** (2.5.2): the node graph is data, the "script" is the DSL, nothing executes as code.

## 6. Assets: bring your own (the Procreate analogy)

A first-class **Asset Library** per group, exactly like importing brushes/fonts into Procreate:

- **Import** fonts (incl. variable fonts — ties to the [`DESIGN.md`](../DESIGN.md) dynamic-type idea), images/graphics, **icon packs** for menus, sounds, later Lottie/motion presets.
- Assets get **stable IDs + semantic names**, so views/themes reference them by name; swapping an asset updates everywhere.
- **Kits** = bundles of assets + tokens + components + rules packaged together (a "WAD?FC neon kit"), shareable via the template gallery — the **Minecraft datapack / Procreate brush set** model.
- **Motion & animation are assets too.** Transitions, springs, loops, and effects are **declarative motion** — motion tokens (durations/curves) plus importable animation files (**Lottie / Rive-style JSON**, keyframe/timeline specs). "Bring your own animation" means bringing a **declarative file**, never a code script — so it stays App-Store-safe (2.5.2, [`AI-READINESS.md`](AI-READINESS.md)) and portable. Everything honors **Reduce Motion** automatically.
- **Guardrails:** imported assets (incl. fonts, images, sounds, motion files) are validated (size, format, licensing reminder), kept slim ([`PERFORMANCE.md`](PERFORMANCE.md): subset fonts, compress images, cap animation cost), and sandboxed as **data** — an asset can never be executable.

## 7. Console: the debugging + observability toolkit (your explicit ask)

Making *feels nice* only if you can see what's happening. The Console gives makers real tools, borrowing from browser devtools + game debug overlays:

- **Live event log** — a running feed: "rule *weekend-poll* fired → created poll, sent 9 notifications", "layout *home* fell back to default (unknown component)". Filterable, timestamped.
- **Inspector** — tap any component in preview → see its type, props, bound data source, which token drives each style, and which layer set it (§8). The "why is this blue?" answer in one tap.
- **Trace / "why did this happen"** — pick an outcome (a notification, a hidden button) and see the chain: trigger → conditions evaluated (with the actual data) → actions. Powered by the rule execution log + the CRDT history.
- **Simulate / dry-run** — fire a fake "event created" or jump the clock to "Saturday 09:00" and watch rules run **without** touching real data. Test before publish.
- **Validation panel** — schema errors, accessibility-floor breaches, perf-budget warnings, surfaced **inline as you edit**, with one-tap fixes.
- **Preview-as** — see the app **as a different role** (member vs admin) or **device** (small phone, huge Dynamic Type, dark mode, Reduce Motion) to catch what others will see.
- **Time-travel** — scrub the history; diff "before/after"; revert any change or any layer. (Free because of the local-first log.)

## 8. Shared "world" vs personal "skin" — the Minecraft client/server-mod model (your explicit ask)

The right mental model is **Minecraft mods**: *server-side* mods change the shared world everyone on the server experiences; *client-side* mods (a shader pack, a UI tweak) change only **your** view and nobody else needs them. We apply that distinction to **aesthetics**:

- **Server-side = the group's shared identity (universal).** What makes the group feel like one place: the WAD?FC **logo, brand color, the Trivselslekene look** — the shared skin everyone sees. Set by the group, synced to all.
- **Client-side = your personal skin (only you).** How *you* want *your* app to look: your color scheme, font + text size, dark/light, density, motion, haptics, your own imported assets. Yours alone; invisible to everyone else.

**The load-bearing principle:** *mechanics and content are always shared; look-and-feel is where the personal layer lives.* The data, the **rules/automations**, the **policies**, and which screens/features exist are the "world mechanics" — there's no personal version of what the group did or how its automations behave, so those stay server-side. Aesthetics are the natural home of client-side freedom.

**The cascade** (layers resolve first-found-wins): **Device → Personal → Group-shared → Default.**

| Layer | Minecraft analogy | Scope | Synced? | Examples |
|---|---|---|---|---|
| **Default** | base game | shipped template | n/a | the baseline everything inherits |
| **Group-shared (published)** | **server-side mod** | whole group | **yes, to all** | logo, brand color, the shared theme/identity; all rules, policies, structure |
| **Group draft** | staging server | editor preview | no (until **Publish**) | work-in-progress before it goes live for everyone |
| **Personal** | **client-side mod** | one member | to that member's devices | my accent, my font, my chosen skin |
| **Device** | local settings | this phone | no | text size, Reduce Motion, haptics |

**The "group can lock it" control (the key nuance).** Just as a Minecraft server can *require* its resource pack, the group can **lock specific aesthetic elements** to protect identity — e.g. the **logo and brand color are enforced for everyone**, but background, accent, font, and density are **left open** for each person to skin. So each aesthetic token is tagged **Shared-locked**, **Shared-default-but-personal-overridable**, or **Personal-only**, and the group decides the mix. Default stance: a strong shared identity with generous personal freedom on top.

**Draft → Publish** gives the "push a server update" feel: an admin edits safely, previews, then publishes the shared layer atomically to everyone (one-tap rollback via the local-first history).

## 9. How this stays shippable (not a 5-year IDE)

We do **not** build all of this at once. Order by value and reuse:

- **v1 (Phase 1):** Depth 1–2 lite — themes/tokens with **live preview**, section show/hide/reorder, **draft→publish**, undo, the **personal/device** override layers, and a basic **Console log + inspector**. This alone feels great and ships with the core loop.
- **Phase 2:** full visual builder (component palette, variants), the **Asset Library** + kits, preview-as-role/device, validation panel.
- **Phase 3:** the **node canvas ↔ script** logic surface, simulate/dry-run, trace, command palette.
- **Throughout:** the **AI** ([`AI-READINESS.md`](AI-READINESS.md)) is the zero-friction on-ramp to every surface — "make it cozier" produces a diff you then refine in the Creator. AI + visual builder + script are three doors to one model.

Each tier reuses the same schemas, capability catalog, validation, and preview — so the Creator grows by *adding surfaces over one model*, never by rebuilding.

## 10. Systems to learn from (for this specifically)

- **Procreate** — asset import + a "studio" for tuning params; brush sets = our kits.
- **Minecraft** — command blocks (command palette), `/gamerule` (params), creative/edit mode, **datapacks** (shareable kits), server vs client.
- **n8n / Node-RED / Unreal Blueprints / Scratch** — node logic with a text counterpart; dual-representation done right.
- **Figma** — inspect, components/variants, edit-on-canvas, "Dev mode"; the gold standard for live, direct manipulation.
- **Webflow / Framer** — visual builder over a real underlying structure (not throwaway).
- **Browser DevTools** — inspector + console + "why is this styled this way"; our Console borrows directly.
- **Cocoon Shell** — a shipping theme builder (live preview + asset packs + exportable bundles + a community store); near-exact proof of pillar 1. ([`FEASIBILITY.md`](FEASIBILITY.md))

## 11. Motion customization + the design file system (import / export)

Two requests that belong together, because both are about **portable, declarative design**:

**Motion & animation, fully customizable.** Everything that moves is parameterized, not hardcoded: **motion tokens** (durations, easing curves, spring stiffness) editable like color tokens with live preview, plus **importable animation files** (Lottie/Rive-style declarative JSON) for richer effects — splash, transitions, reactions, the Trivselslekene flourishes. Power users tune curves on a timeline; everyone else picks a motion preset. It's all **data, never code** (so it's App-Store-safe and survives to a future Android renderer), and the engine enforces **Reduce Motion** + a per-frame cost budget ([`PERFORMANCE.md`](PERFORMANCE.md)) so a fancy animation can't make the app janky.

**A design file system — import & export.** A whole look (theme + tokens + motion + assets + layout + rules) is a **bundle**: a manifest (JSON) plus its assets. You can:
- **Export** your group's (or your personal) design as a portable bundle — to the iOS **Files app**, the share sheet, AirDrop, or a link (assets ride in R2 [`FEASIBILITY.md`](FEASIBILITY.md)).
- **Import** a bundle from Files / a link / a friend — it's **validated against the schema + registry** before it can apply, and previewed as a diff before you accept ([`AI-READINESS.md`](AI-READINESS.md) loop).
- **Browse a local library** of saved designs/kits on-device, apply/duplicate/edit — the Procreate/Cocoon model.

This is what makes designs shareable **without** us running a store: bundles move peer-to-peer or via a third-party host. A public store, if ever wanted, can be **third-party** (the bundle format is open) — we don't have to build or moderate it ourselves. **Safety:** an imported bundle is pure data; it can't carry code; assets are validated; rules inside it are shown for review before they run ([`RULES-SPEC.md`](RULES-SPEC.md)).

## 12. Open questions

- How much of Depth 2 is **edit-on-the-real-screen** vs a side panel? (Edit-in-place feels best but is harder.)
- Do **personal overrides** risk fragmenting the group's shared identity — should the group be able to lock certain tokens? (Yes — see §8; open question is the *defaults*.)
- Node canvas on a **phone screen** is cramped — is Logic primarily an iPad/landscape surface, or do we design a phone-first node UX?
- Kit **trust/scanning** before import (shared assets/rules from other groups) — ties to the gallery moderation question.
- Where's the line between "policy" (Group layer config) and "code" we must keep out (2.5.2)? (Policies are declarative params; keep them so.)

---

### Related
[`VISION.md`](VISION.md) · [`SDUI-SPEC.md`](SDUI-SPEC.md) (views) · [`RULES-SPEC.md`](RULES-SPEC.md) (the logic documents) · [`AI-READINESS.md`](AI-READINESS.md) (talk-it-into-being on-ramp) · [`PERFORMANCE.md`](PERFORMANCE.md) (asset/preview budgets) · [`REALTIME.md`](REALTIME.md) (history = time-travel) · [`FEASIBILITY.md`](FEASIBILITY.md).
