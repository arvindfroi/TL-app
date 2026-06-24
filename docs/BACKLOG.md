# Feature backlog

> **Re-cut for the customization pivot (June 2026).** See [`PIVOT.md`](PIVOT.md). Seed list for GitHub Issues, grouped by the new phases.

## Phase 0 — Platform foundations
- [ ] Register Apple Developer Program
- [ ] CloudKit container; **CKShare zone-per-group** model (not one hardcoded zone)
- [ ] Sign in with Apple (one user, many groups)
- [ ] **Group entity**: create / leave / delete a group
- [ ] **Roles & permissions (RBAC)**: owner / admin / member / invited, enforced in data layer
- [ ] **Invites**: invite links / codes, accept / revoke
- [ ] App scaffold (SwiftUI); seed WAD?FC as the first group via the normal flow
- [ ] Cloudflare account + R2 bucket reserved for media
- [ ] TestFlight setup (early phases)

## Phase 1 — Theming + the core loop (default template)
- [ ] **Theme-token system** (color roles, type scale + variable-font axes, spacing, radii, motion) + accessibility floors
- [ ] Group chat (text + images) + read receipts, built on tokens
- [ ] Push notifications via CloudKit subscriptions
- [ ] Events: create, RSVP (going/maybe/no), comments
- [ ] Polls: question, options, live results
- [ ] "Hjemme i Sandnes" calendar with group overlap view
- [ ] In-app **theme editor** with live preview (the friendly customization v1)

## Phase 2 — SDUI layout engine
- [ ] **Layout document schema** + versioning + graceful fallback ([`SDUI-SPEC.md`](SDUI-SPEC.md))
- [ ] **Component registry** (the data-not-code safety boundary)
- [ ] **Data-binding catalog** (read through the permission layer)
- [ ] **Renderer**: document → native SwiftUI
- [ ] **Friendly editor**: reorder/toggle sections, pick component variants
- [ ] **Advanced editor**: raw document editing + validation + preview + rollback (RBAC-gated)

## Phase 3 — Rules / automation engine
- [ ] **Rule schema** (trigger → condition → action) + validation ([`RULES-SPEC.md`](RULES-SPEC.md))
- [ ] **Evaluator** (on-device) + scheduled rules via serverless cron
- [ ] **Dedup / claim record** so a rule runs once across all devices
- [ ] Fixed **action allow-list** (notify, postMessage, createPoll, awardPoints, …)
- [ ] **Friendly rule builder** + templates
- [ ] **Advanced rule editor** + dry-run/simulate + execution log + kill switch (RBAC-gated)

## Phase 4 — Public release + moderation + sharing
- [ ] **Guideline 1.2 UGC**: EULA, in-app **report**, **block**, content hold/hide, **age-gating (1.2.1)**
- [ ] Admin moderation tools (per-group) + backstop report path
- [ ] Privacy disclosures for customization/automation; no cross-group data sharing
- [ ] **Template gallery**: export/import themes, layouts, rules; review/scan before an imported template/rule runs
- [ ] App Store submission prep (data-not-code review notes for 2.5.2)

## Phase 5 — Modules on top of the platform (carried over)
- [ ] Daily vlog / BeReal (ephemeral; files on R2) — as an editable module
- [ ] "Who's it today" client-side selection
- [ ] Ønskelister + Secret Santa · TL bucket list
- [ ] Home Screen widget / Live Activity / Siri (must read theme + layout)
- [ ] Apple Intelligence: smart replies / summaries (gated)
- [ ] **Trivselslekene module**: point system, leaderboard, trophies, live scoring, history, player animations
- [ ] TL ferien 2030: shared savings goal/tracker (no money held)
- [ ] "TL polymarket": points-only first (legal note in roadmap §6)

## Open decisions
- [ ] Customization editor depth for v1 (drag-drop vs. section toggles before raw editing)
- [ ] Which triggers/actions ship first in the rules engine
- [ ] Template gallery trust model + abuse-scanning of imported templates/rules
- [ ] Confirm "friend group" size assumption (≤ ~50) for CloudKit quota economics
- [ ] Moderation backstop visibility vs. the private-zone privacy promise
- [ ] Vlog lifespan + mode (one random person/day vs everyone/day)
