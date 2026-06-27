# TL-appen — the plan

*The single source of truth for what we're building. Last updated June 2026. Status: idea / pre-development — no app code yet, this is the thinking.*

> **"TL-appen" is a working codename.** The project is moving from a single-group app to a product any friend group can use; a real brand name is still to be decided.

If anything elsewhere disagrees with this file, **this file wins.** The earlier, detailed working docs (engine specs, prior-art, the original brainstorm, the full decision log) are kept in a local `archive/` folder for reference only — they are superseded and not part of this repo.

---

## 1. What we're building

A **local-first, deeply customizable space for a friend group** — less an app you configure, more a clubhouse you and your friends keep making your own. It does three jobs well:

1. **Plan hangouts** — where, when, who, what (events + RSVP + polls + a "who's home" availability calendar).
2. **Group chat** — one shared space, instant, with read receipts.
3. **Trivselslekene** — the yearly games event: scoring, leaderboard, trophies, history.

It's built for one real friend group first — the design partner — then opened to any group. Any friend group can create their own **private, invite-only** group. It's multi-tenant — but a network of *private islands*, **not** a public social platform. The three core jobs ship as a great default that any group can then reshape.

**Total customization is the heart of the product.** People must be able to **import/export** and **write scripts/config** that control how the app looks and feels — essentially everything: layout, assets, fonts, colors, animations/motion, rules, logic, buttons, components, icons, sounds, and more. You change what you want and share it as a portable file; the rest inherits a beautiful default. (App-Store-safe because it's all declarative data, never executed code — see [D4](#2-the-decisions-that-are-locked) and §5.)

**Target feel:** Cocoon-Shell-style UI customization + Minecraft-command-block automations + Snapchat-feel chat — running free on Apple's stack.

---

## 2. The decisions that are locked

These are settled. Only monetization is deferred.

| # | Decision | What it means |
|---|---|---|
| **D1** | **Private groups, not a public platform** | Groups are invite-only. No public feed, discovery, stranger-mixing, or public template store in v1. So no at-large UGC moderation / age-gating either — those only come back *if* we ever add a public surface (not planned). |
| **D2** | **Multi-group, Instagram-style switching** | One Sign in with Apple identity; a per-group profile (your name/avatar can differ per group); a group switcher with one active group at a time. **No data crosses groups.** |
| **D3** | **iOS-only — Android not pursued** | No Skip/Android investment. We keep one cheap discipline (a "thin seam": the core logic imports no SwiftUI and no CloudKit) **only** for testability and a possible backend swap — *not* for Android. |
| **D4** | **Customization is data, never code** | All customization — layouts, automations, themes, motion, imported assets — is declarative data interpreted by engines baked into the app. App Store **2.5.2**-safe; not the HTML5/JS webview route (**4.7**). It's *React-like + CSS-like, rendered natively*. |
| **D5** | **Settings/editor-first; AI comes later** | v1 customization is a visual editor + token editing + paste-config/script power surface. The "talk the app into being" AI is a **later** bet; v1 only *futureproofs* for it (everything is validated declarative documents = a clean schema-as-API). |
| **D6** | **One model, separated modules** | Conceptually one object graph, but the View renderer, Reaction evaluator, and Theme resolver are **independent modules** behind interfaces — not one mega-renderer. (This is the deliberate guard against the Spotify-HubFramework over-abstraction trap.) |
| **D7** | **Spikes before product code** | The novel parts (local-first store, SDUI perf) get throwaway prototypes first. The CRDT approach has a **named Plan B**: if it proves uncontrollable, fall back to CloudKit-as-source-of-truth + optimistic local updates. |

**Non-negotiable, every phase and every PR:** privacy by default · **no ads, ever** · data-not-code · **~$0 hosting**.

**Deferred (not v1):** monetization (private + small groups stay on free tiers, so "no ads" survives), a public template store, conversational-AI customization, Android.

---

## 3. Architecture

**Principles:** local-first (each device holds the truth, cloud is transport); one domain-typed model with views/reactions/looks as facets; data-not-code; light trust inside a group; iOS-native with a portable core; ~$0 hosting.

**The stack, bottom to top:**

1. **Identity** — Sign in with Apple. One person, many groups.
2. **Local-first store (the source of truth)** — an event-sourced / CRDT operation log on each device. Instant, offline, conflict-free merge. Growth is bounded by snapshots + compaction. Built on a pure-Swift core + a proven CRDT library.
3. **Transport & blobs (not the brain)** — **CloudKit** moves changes between devices and delivers push; **Cloudflare R2** holds media/assets (10 GB free, zero egress).
4. **One object model** — domain-typed objects (`Event`, `Poll`, `Message`, `Tradition`, `Member`…) whose **views** (layout), **reactions** (automations), and **looks** (theme/motion) are facets.
5. **Engines in the binary** — a view renderer (data → native SwiftUI), a reaction evaluator, and declarative motion. Compile-once for speed. Only *data* is ever downloaded.
6. **Creator + AI** — the visual/logic/asset/console editor and (later) the AI assistant, both clients of one validated **propose → preview → approve** edit API.
7. **Integration** — the default app, design import/export via the Files app, and deep Apple integration: App Intents/Siri, Widgets, Live Activities, on-device Foundation Models.

**Portable core (ports & adapters).** The brain — model, view compiler, reaction evaluator, validation, catalogs, permission logic — is pure Swift with no SwiftUI and no CloudKit. Everything Apple-specific sits behind protocols (`Store`, `SyncEngine`, `BlobStore`, `Push`, `AIProvider`) with Apple adapters. Consequence: getting faster = improve an adapter; swapping the backend = one new adapter. (We keep this for cleanliness and the CRDT Plan-B swap, not for Android.)

**Trust model.** Inside a group it's a handful of friends who trust each other → light, social permissions and generous undo, not an enterprise permission matrix. The only guarded edge is **imported design bundles**, which are validated-as-data and reviewed-before-run.

**Data flow.** Your action applies instantly on-device; CloudKit push propagates to others in ~1–2s. Two lanes: **durable** state (messages, events, polls, points) in the synced log, and **ephemeral** signals (typing, presence) over a fast lossy channel that's never stored. Automations are deduplicated so a rule runs once across all devices, not once per device. Unknown components/reactions from a newer app version degrade gracefully — they never crash an older version.

---

## 4. The data model (the foundation)

One graph, domain-typed. Every object shares an **envelope** (`id`, `type`, `schema` version, `group`, `author`, timestamps, `deleted` tombstone, typed `fields`) and a typed body.

- **IDs:** ULID — sortable, globally unique, generated offline with no server round-trip.
- **Ordering:** Hybrid Logical Clocks (HLC) give a deterministic total order across devices without trusting device clocks.
- **State = fold of operations.** Each device keeps an append-only op log per group; materialized state is the fold. New ops arrive via sync and merge by HLC. Growth is bounded by snapshots + compaction.

**Conflict resolution — CRDT kind per field** (this is what makes "two people edit offline" safe):

| Field kind | Used for | Merge rule |
|---|---|---|
| LWW-Register | scalars (title, date, RSVP status, a theme token) | last write wins by HLC; loser kept in history |
| OR-Set (add-wins) | sets (poll options, tags, members, availability days) | concurrent adds survive; remove only cancels observed adds |
| PN-Counter | points / tallies | per-member increments summed — never clobbered |
| Append log | chat messages | immutable, add-only, HLC-ordered; edits = a new LWW `editedBody` field |
| Ordered list | a view's child order | v1: explicit LWW `order` array; later: fractional indexing if needed |
| Tombstone | deletion | mark `deleted`; compaction removes later |

**Object types (v1 vocabulary).** Domain: `Group`, `Member`, `Message`, `Event`, `RSVP`, `Poll`, `Vote`, `AvailabilityDay` ("who's home"), `PointsEntry`, `Tradition` (a recurring, optionally-scored ritual with history). Customization facets: `View` (layout), `Reaction` (automation rule), `Theme` (tokens + motion), `AssetRef` (pointer to an R2 blob by content hash).

**Identity & multi-group:** one Apple identity per person; one `Member` record per (person, group) with a per-group profile; the app renders only the active group's zone. No data crosses groups.

**Storage mapping:** operations → CloudKit records in the group's CKShare zone (push via CloudKit subscriptions); blobs → R2 by content hash (objects hold an `AssetRef`, never the bytes); optional client-side E2EE so CloudKit/R2 hold ciphertext. The `Store`/`SyncEngine` ports keep this mapping to a single adapter.

**The walking skeleton:** don't build all types at once. The first end-to-end slice is just `Group` + `Member` + `Message` over the local-first op log, rendered by one hardcoded view, synced via CloudKit — proving create-group → invite → offline-message → merge.

---

## 5. The Creator — making it yours, without overwhelm

**The full scope of what's customizable (a core requirement):** layout/screens, components and buttons, fonts (incl. variable-font axes), colors, spacing/radii, icons, sounds, assets (bring your own), animations/motion, rules/automations, logic, and policies — all of it editable, **scriptable**, and **importable/exportable** as portable bundles. Anything the app shows or does should be reachable as declarative data. Nothing in this list is hardcoded-only.

When a group can change all of that, that's hundreds of knobs. The Creator tames that with five principles: **sparse overrides over a complete default** (you change the 3 things you care about; 397 inherit), **tokens + cascade** (change a semantic token once, it ripples), **progressive disclosure** (four depth doors, most people stop at the first two), **safe by construction** (accessibility + performance are live guardrails you can't violate), and **always-live, always-reversible** (every change previews on the real app; the local-first history is unlimited undo / time-travel).

**Four creation depths:** **Pick** (swap a theme, drop in a font/icon pack, flip toggles) → **Arrange** (visual builder, drag sections on the live screen) → **Wire** (node canvas: trigger → condition → action) → **Script** (the raw rule/document text + command palette). Wire and Script are two views of the same document; nothing is second-class.

**One Workshop, four surfaces:** **Design** (visual builder + token editor), **Logic** (node canvas ↔ script for automations), **Assets** (bring-your-own fonts/images/icons/sounds/motion files, packaged as "kits"), and **Console** (live event log, inspector, a "why did this happen?" trace, simulate/dry-run, validation warnings, preview-as-role/device, time-travel).

**Shared vs personal — the Minecraft mod model.** *Mechanics and content are always shared; look-and-feel is where personal freedom lives.* The group's shared identity (logo, brand color, the Trivselslekene look) is "server-side" and syncs to everyone; your personal skin (your accent, font, density, dark/light, motion) is "client-side" and yours alone. The cascade resolves **Device → Personal → Group-shared → Default**. The group can **lock** specific tokens (e.g. enforce the logo and brand color) while leaving the rest open to personal skinning. A **draft → publish** control gives admins the "push a server update" feel, with one-tap rollback.

**Motion is data too.** Motion tokens (durations, curves, springs) are edited like color tokens; richer effects are importable declarative animation files (Lottie/Rive-style JSON) — never code, so it stays App-Store-safe and portable, and always honors Reduce Motion + a per-frame cost budget.

**Import/export.** A whole look (theme + tokens + motion + assets + layout + rules) is a portable **bundle** (a JSON manifest + assets). Export to Files / AirDrop / a link; import is validated against the schema and previewed as a diff before it applies. This makes designs shareable **without us running a store** — a public store, if ever wanted, can be third-party since the format is open.

---

## 6. Cost — why it's genuinely ~$0

| Need | Service | Free allowance |
|---|---|---|
| App data, chat, configs, rules, sync, push | **CloudKit** | 10 GB asset + 100 MB DB **+250 MB/user**, 2 GB/day transfer, free push |
| Photos / video / theme assets | **Cloudflare R2** | 10 GB storage, **zero egress** |
| Scheduled automations ("every Monday", "1 day before an event") | **Free serverless cron** (Cloudflare Workers / Val Town) | trivial load |
| Identity | **Sign in with Apple** | free |
| **Only recurring cost** | **Apple Developer Program** | **~$99/yr** — unavoidable to ship to iPhones |

The CKShare model means most storage rides each user's own iCloud quota, which is what lets "no ads, ever" survive even as groups are added. The two things that could ever push past free — a *public* theme store and *sub-second* always-on presence — are both deferred and both have free-enough workarounds (peer-to-peer bundle sharing; low-fidelity presence over push). Neither blocks v1.

---

## 7. Roadmap

Loose, hobby-pace, TestFlight-first. Each phase ships a usable increment.

**Phase 0 — Foundations & de-risking.** Apple Developer Program + CloudKit container; the multi-group data model + roles + invites + Sign in with Apple; app scaffold; R2 bucket reserved. **Run the spikes first** (see §8). *Outcome: a person can create a group, invite a friend, they join — both see an empty themed shell.*

**Phase 1 — MVP core loop + theming (the real launch).** Group chat + read receipts → events + RSVP → polls → "who's home" calendar → push — all built **on top of theme tokens + live preview**, so the default look is already restylable. Plus Creator v1-lite: themes/tokens with live preview, section show/hide/reorder, draft→publish, undo, personal/device overrides, a basic Console log + inspector. *Outcome: the group actually plans hangouts in it.*

**Phase 2 — Social & iOS flavor + visual builder.** TL daily vlog/BeReal (ephemeral), wishlists + Secret Santa, the TL bucket list; Home Screen widget, a Live Activity, a Siri shortcut, opportunistic Apple Intelligence (smart replies, summaries) gated to capable devices. Creator: full visual builder (component palette + variants), the Asset Library + kits, preview-as, validation panel. *Outcome: fun to open daily.*

**Phase 3 — Trivselslekene & gamification + rules engine.** Points + leaderboard, trophies/achievements/records, the dedicated **Trivselslekene module** (live scoring, history, the player animations). Creator: the node-canvas ↔ script Logic surface, simulate/dry-run, trace, command palette. *Outcome: the app owns the group's signature tradition.*

**Phase 4 — Money (with care).** **TL ferien 2030**: a shared savings goal + tracker + planning board (the app holds no money). **Vipps "TL polymarket"**: launch **points-only / non-monetary first** — Norwegian gambling law means the app must never hold, route, or take a cut of real money.

*Deferred bets layered on later: conversational-AI customization, and any public template sharing.*

---

## 8. De-risking spikes (do these in Phase 0, timeboxed, disposable)

The engines are the only genuinely novel parts; everything else (chat/events/polls on CloudKit) is well-trodden. Each spike answers one yes/no question, then the code is thrown away.

- **Spike L — local-first slice.** A CRDT log of "messages" on two devices, merging offline edits, CloudKit purely as transport. *Does it feel instant? Does merge behave?* (If no → Plan B: CloudKit as source of truth + optimistic updates.)
- **Spike A — SDUI render performance.** Hand-compile one real screen and prove 60/120 fps scroll + the cold-start budget on the oldest target device. *Is customization inherently janky?*
- **Spike B — multi-CKShare tenancy.** One user in two groups, data isolated, invite/accept works. *Validates the core architecture shift.*
- **Spike C — reaction dedup.** A claim-record gives exactly-once execution across 3 simulated devices. *Validates automation correctness.*
- **Spike D — customization fallback.** A corrupt/newer layout degrades to default without crashing. *Validates the safety story.*

---

## 9. Risks & open questions

**Top risk — local-first complexity.** CRDTs add merge/tombstone/explainability cost. Tamed by: starting with simple, well-understood CRDT kinds (covers the whole MVP); bounding the log with snapshots; keeping merges *legible* (every field shows who last set it and when, with a history scrubber); and the named Plan B above. Local-first is the goal, not a religion.

**Over-engineering / never shipping.** Guarded by the "sparse overrides over a default" Creator, the walking-skeleton-first build order, domain-typed (not generic) components, and shipping the default template as real `EventCard`/`PollCard` components rather than a soup of primitives.

**Performance is a gate, not a phase.** Every PR meets the budgets; customization compiles once and the hot path is plain SwiftUI; a slow feature gets redesigned, not shipped.

**Norwegian gambling law.** Informal betting between friends is fine; an app that holds/routes money or takes a cut risks being treated as a commercial operator. Safe path: points-only; if real money is ever attached, settlement stays strictly peer-to-peer in Vipps with the app taking nothing — and get advice first.

**Bus factor.** The CKShare zone has one owner; document who owns it and keep periodic exports.

**Still open:** ordered-list strategy (LWW array vs fractional indexing); message edit/delete semantics; how much of the visual builder is edit-on-the-real-screen vs a side panel; whether the node canvas is iPad-first or phone-first; and the exact line between declarative "policy" and code we must keep out (keep policies as declarative params).

---

## 10. Working in this repo

**Status:** idea / pre-development. No app code yet — this repo is the product + architecture thinking. When code lands, the Xcode project / Swift packages go under a top-level app folder.

**Guardrails for every contribution (human or AI):**

- **Privacy by default** — local-first/own-your-data, Sign in with Apple, minimum data, encrypted in transit + at rest, permission checks in the data layer (not just UI). No secrets in the repo (`.gitignore` covers `*.p8`, `*.p12`, `*.mobileprovision`, `.env`).
- **No ads, ever** — no ad SDKs, slots, tracking, or sponsored content.
- **Data, not code** — all customization is declarative data interpreted by in-binary engines (App Store 2.5.2); never download/execute code, never the webview mini-app route (4.7). "Bring your own" = a declarative file, not a script.
- **~$0 hosting** — no feature that needs a paid always-on server.
- **Keep the portable core clean** — model/engines/validation import no SwiftUI and no CloudKit; Apple specifics live behind ports/adapters.
- **Don't hardcode any group or member names** — every group, including the first, is created via the normal create/invite flow.
- **Betting** never holds/routes money or takes a cut — points-only first.
- **Accessibility floors** (Dynamic Type, contrast, Reduce Motion) and **performance budgets** are enforced in the engines, so a custom theme/motion can't make the app unusable or janky.

**Stack at a glance:** Swift / SwiftUI (iOS-only) · Sign in with Apple · local-first CRDT core · CloudKit (sync transport + push) · Cloudflare R2 (media/assets) · free serverless cron (scheduled automations) · on-device Apple Foundation Models (opportunistic) · Vipps (peer-to-peer only, later).

---

*Deeper background — the original brainstorm, the engine specs (SDUI / rules / realtime / security / performance), prior-art notes, and the full decision log — is preserved in a local `archive/` folder (kept on disk, not pushed to GitHub). It's reference material; this file is the current plan.*
