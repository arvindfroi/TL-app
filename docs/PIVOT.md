# PIVOT — from "one private app for us" to "a customizable app any friend group can make their own"

> **Status: proposed pivot (June 2026).** This document is the master plan for the pivot. It supersedes the *scope and architecture* assumptions in the original [`MVP-and-Roadmap.md`](MVP-and-Roadmap.md) and [`ARCHITECTURE.md`](ARCHITECTURE.md), which were written for a single hardcoded group. The three jobs the app does (plan hangouts, group chat, Trivselslekene) stay — *how* it's built changes a lot.

---

## 1. What changed, in one paragraph

We are no longer building a fixed app for nine named friends. We are building **a platform any friend group can adopt and reshape to fit their own needs and aesthetic** — with **admins and invites**, **multiple independent groups**, **user-editable layouts and design (not just colors/fonts)**, and an **advanced "rules/scripts" layer** for power users. Our own crew (WAD?FC) becomes the first tenant and the design partner, not the hardcoded universe. It stays **iOS-only for now** (deep Apple integration), **ad-free forever**, and **privacy-first** — but it is now a **public, multi-tenant App Store app**, which pulls in real obligations the old TestFlight-only plan let us skip.

## 2. The four decisions that drove this rewrite

| Decision | Choice | Consequence |
|---|---|---|
| **Who it's for** | **Any friend group** — each with its own needs and aesthetic | Multi-tenant data model, real roles/invites, public App Store release, moderation duties |
| **Customization ceiling** | **Full server-driven UI (SDUI) + a rules/automation engine** | Two new in-house engines; screens and behavior become *data*, not hardcoded Swift |
| **Platforms** | **iOS-only for now** | Keep the engines Apple-native; design them portable so Android (Skip) stays possible later without a rewrite |
| **Deliverable** | **Both** a standalone pivot plan and updated repo docs | This file + rewritten ARCHITECTURE/README/AGENTS/DESIGN/roadmap/backlog, plus two engine specs |

## 3. What the old plan assumed — and why each assumption breaks

The original architecture leaned on four simplifications that the pivot removes:

**"Membership *is* the CloudKit shared zone; no roles, no allowlist."** That works for exactly one group with everyone equal. The moment there are *many* groups, **admins**, and **invites**, we need a real membership model: a group entity, members with roles, and an invite mechanism. CKShare still does the heavy lifting for *access* (a participant either can reach a group's data or can't), but **roles and permissions become an app-level concept layered on top** of CKShare — see [`ARCHITECTURE.md`](ARCHITECTURE.md).

**"Screens are hardcoded SwiftUI."** Full customization means a screen is no longer Swift we ship — it's a **declarative layout document** the group can edit, rendered at runtime by an engine baked into the binary. This is the single biggest change. Spec: [`SDUI-SPEC.md`](SDUI-SPEC.md).

**"Behavior is hardcoded."** "Advanced settings / rules scripts" means groups can define **triggers → conditions → actions** ("when an event is created on a weekend, post a poll for the venue and notify everyone"). This is a **sandboxed, declarative automation engine**, never downloaded code (App Store 2.5.2). Spec: [`RULES-SPEC.md`](RULES-SPEC.md).

**"TestFlight-only, so we skip App Store UGC review."** A public app any group can download **must** satisfy Guideline 1.2: an EULA, in-app **report and block**, content moderation/filtering, and age-gating. This is now a first-class workstream, not a someday-if-we-go-public note.

## 4. The new architecture in one screen

Seven layers, bottom to top. Each is detailed in [`ARCHITECTURE.md`](ARCHITECTURE.md); the two new engines have their own specs.

1. **Identity** — Sign in with Apple. One person, many groups.
2. **Tenancy & access** — each **group** is its own CloudKit **CKShare** shared zone. A user *participates* in every group they belong to. The zone boundary is the hard security wall.
3. **Roles & permissions (app-level RBAC)** — `owner` / `admin` / `member` / `invited`, stored per group, enforced in the data layer (not just hidden in the UI). Defines who can edit layouts, who can write rules, who can moderate.
4. **Domain data** — the content entities: messages, events, RSVPs, polls, availability, points, plus media metadata (files still on **Cloudflare R2**).
5. **Customization store** — the per-group **theme tokens**, **layout documents** (SDUI), and **rule definitions**, versioned, with a safe fallback to defaults.
6. **Engines (new, embedded in the binary)** — the **Layout/SDUI renderer** (data → native SwiftUI) and the **Rules evaluator** (declarative automation). Both are interpreters shipped *inside* the app; only *data* is downloaded.
7. **Experience & integration** — the default app experience, the **in-app customization editor** (friendly + advanced modes), the **template gallery** (share/import group setups), and Apple integration (Widgets, Live Activities, Siri) that must also read theme/layout.

## 5. The two new engines (why they're the heart of the pivot)

**Layout / SDUI engine.** Screens are described by a versioned JSON document: a tree of *components* (from a fixed registry baked into the app — chat list, event card, poll, leaderboard, header, stack, grid, etc.), each bound to domain data and styled by theme tokens. The renderer walks that tree and produces native SwiftUI. Friendly mode = a drag-and-drop/section editor; advanced mode = direct document editing. Unknown or future components **degrade gracefully** so an old app version never hard-crashes on a newer layout. Full schema, component catalog, theming, and editor design in [`SDUI-SPEC.md`](SDUI-SPEC.md).

**Rules / automation engine.** A group defines automations as **trigger → condition → action** records in a small, sandboxed DSL (think Shortcuts/IFTTT, expressed as data). Triggers fire on data changes (event created, RSVP changed, member joined) or on a schedule (a date is reached, every Monday). The evaluator is deterministic, resource-bounded, and side-effect-limited to a fixed allow-list of actions (notify, post message, create poll, award points, toggle a setting). Because many devices sync the same data, rule execution is **deduplicated/claimed** so an action runs once, not nine times. Scheduled rules use the existing free serverless cron. Full DSL, evaluator model, and safety bounds in [`RULES-SPEC.md`](RULES-SPEC.md).

**The hard App Store constraint for both:** Guideline **2.5.2** forbids downloading and executing code that changes app functionality. Our engines are **interpreters embedded in the binary that consume declarative data** — this is the established, compliant pattern (and deliberately *not* the HTML5/JavaScript "mini-app" path, which would drag us into Guideline 4.7's index/disclosure/native-bridge regime). Layouts and rules are content, not code.

## 6. New obligations a public multi-tenant app takes on

Going public is the cost of "any friend group." None of these existed in the TestFlight-only plan:

- **UGC compliance (Guideline 1.2):** an EULA accepting a no-objectionable-content policy, in-app **Report** and **Block**, a moderation path (hide/hold flagged content), and **age-gating (1.2.1)**.
- **Moderation at the group level:** each group's **admins** are its first-line moderators (powers granted by the RBAC layer); we provide the tools and a backstop reporting path.
- **Per-tenant safety of customization:** a malicious or broken layout/rule must never escalate privileges, exfiltrate another group's data, or brick the app. The zone boundary (CKShare) + sandboxed engines + graceful fallback are the defenses.
- **Privacy disclosure:** customization and any automation must declare data use; no cross-group data sharing without consent. (Note current App Store rules on AI/third-party data sharing if we ever add a cloud model — keep on-device Foundation Models.)
- **Cost & sustainability while staying ad-free:** CloudKit storage and traffic largely sit in *each user's own iCloud quota*, so per-tenant cost stays near-zero even as group count grows — this is what lets "no ads, ever" survive a public release. R2 media stays light because vlogs expire.

## 7. What carries over unchanged

The product soul and most of the stack survive: **Swift / SwiftUI**, **CloudKit + R2**, **CloudKit-subscription push (no server)**, **free serverless cron** for scheduled jobs, **Sign in with Apple**, **opportunistic on-device Apple Intelligence**, and the **non-negotiables — privacy by default and no ads, ever.** The three core jobs (plan hangouts, group chat, Trivselslekene) are still the point; they just become the *default template* that ships with the app, which any group can then reshape.

## 8. Revised roadmap (phases re-cut around the pivot)

The old phases assumed a fixed app. The new sequence front-loads the platform foundations the customization story depends on, then layers features as editable templates. Full detail in [`MVP-and-Roadmap.md`](MVP-and-Roadmap.md); summary:

- **Phase 0 — Platform foundations.** Apple Developer + CloudKit container; **multi-group data model**, **roles & invites**, Sign in with Apple, R2 reserved, app scaffold. Outcome: a person can create a group, invite members, and join — with no features yet.
- **Phase 1 — Theming + the core loop as a default template.** Ship chat, events/RSVP, polls, the "home in Sandnes" calendar, push — but built **on top of the theme-token system** so the default look is already restylable. Outcome: a usable app whose colors/type/spacing/component variants a group can change. **This is the real launch.**
- **Phase 2 — The SDUI layout engine + friendly editor.** Screens become layout documents; ship the renderer, the component registry, and the **friendly customization editor** (rearrange/add/remove sections). Advanced raw-document editing behind a flag. Outcome: groups change *layouts*, not just styles.
- **Phase 3 — The rules / automation engine.** Trigger→condition→action DSL, the evaluator, dedup, scheduled rules, and a friendly rule builder + advanced editor. Outcome: "advanced settings" power users automate their group.
- **Phase 4 — Public release + moderation + template gallery.** Guideline 1.2 workstream (EULA, report/block, moderation, age-gating), then the **template gallery** so groups share/import setups. Outcome: open to any friend group.
- **Phase 5 — Trivselslekene, gamification, iOS extras, money (with care)** as editable modules/templates, carrying over the original Phase 2–4 ideas on top of the platform.

## 9. Migration path from where the repo is today

The repo is **pre-code** — there's no app to refactor, only docs and a backlog, which is the cheapest possible moment to pivot. Migration is therefore mostly *re-planning*, not code surgery:

1. **Adopt the new docs** (this file + rewritten ARCHITECTURE/README/AGENTS/DESIGN + the two engine specs + revised roadmap/backlog).
2. **Re-cut the GitHub Issues/backlog** to the new phases; the old phase labels still map (foundations → platform foundations, etc.) but several new issues appear (roles/invites, theme tokens, SDUI renderer, rules engine, moderation, template gallery).
3. **Build Phase 0 against the multi-tenant model from day one** — never hardcode the nine names again; seed WAD?FC as the first group via the same create/invite flow every group will use.
4. **Treat the original three features as the default template**, authored in the SDUI/theme system as soon as Phase 2 lands, rather than as bespoke screens.

## 10. Open decisions (carry into Issues)

- **Editor depth for v1 of customization:** how much drag-and-drop vs. structured section toggles before we expose raw document editing?
- **Rules surface area:** which triggers/actions ship first? (Recommend the smallest safe set, expand by demand.)
- **Template gallery trust model:** are shared templates curated, community-rated, or free-for-all — and how do we scan an imported template/rule for abuse before it runs?
- **Group size & quota assumptions:** "friend group" implies small (≤ ~50); confirm so we keep the CloudKit-per-user-quota economics valid.
- **Moderation backstop:** with everything in private CKShare zones, what is *our* visibility/recourse for an abuse report we legally must act on, without breaking the privacy promise?

---

### Sources (App Store constraints verified June 2026)
- [App Review Guidelines — Apple Developer](https://developer.apple.com/app-store/review/guidelines/) (2.5.2 interpreted code; 4.7 mini apps; 1.2 / 1.2.1 UGC)
- [Apple's Guideline 4.7 update — what HTML5/mini-app hosts must know](https://dev.to/arshtechpro/apples-guideline-47-update-what-every-developer-hosting-html5-mini-apps-must-know-90)
- [App Store Guideline 1.2 — Safety / User Generated Content (BuddyBoss)](https://buddyboss.com/docs/app-store-guideline-1-2-safety-user-generated-content/)
- [Apple expands list of UGC experiences removable without notice (MacTech, Feb 2026)](https://www.mactech.com/2026/02/06/apple-updates-its-app-review-guidelines-with-expanded-list-of-apps-with-objectionable-content/)
