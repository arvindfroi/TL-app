# PIVOT — from "one private app for us" to "a customizable app any friend group can make their own"

> 🕰️ **HISTORICAL (June 2026).** This was the *first* pivot — single-group → multi-tenant. The direction has since evolved further to a **local-first, single-model "group OS"**: read [`VISION.md`](VISION.md) and [`ARCHITECTURE.md`](ARCHITECTURE.md) for current truth. Kept because the **multi-tenant, data-not-code, and App-Store reasoning here still holds**; only the *framing* moved on.
>
> **Status (original):** proposed pivot. Superseded the *scope* assumptions in [`MVP-and-Roadmap.md`](MVP-and-Roadmap.md). The three jobs (plan hangouts, group chat, Trivselslekene) stay — *how* it's built changed.

---

## 1. What changed, in one paragraph

We are no longer building a fixed app for nine named friends. We are building **a platform any friend group can adopt and reshape to fit their own needs and aesthetic** — with **admins and invites**, **multiple independent groups**, **user-editable layouts and design (not just colors/fonts)**, and an **advanced "rules/scripts" layer** for power users. Our own crew (WAD?FC) becomes the first tenant and the design partner, not the hardcoded universe. It stays **iOS-only for now**, **ad-free forever**, and **privacy-first** — but it is now a **public, multi-tenant App Store app**, which pulls in real obligations the old TestFlight-only plan let us skip.

## 2. The four decisions that drove this rewrite

| Decision | Choice | Consequence |
|---|---|---|
| **Who it's for** | **Any friend group** | Multi-tenant data model, real roles/invites, public release, moderation duties |
| **Customization ceiling** | **Full server-driven UI (SDUI) + a rules/automation engine** | Two in-house engines; screens & behavior become *data* |
| **Platforms** | **iOS-only for now** | Keep engines Apple-native; design them portable so Android (Skip) stays possible |
| **Deliverable** | **Both** a standalone pivot plan and updated repo docs | This file + the rest of the doc set |

*(For current architecture, see [`VISION.md`](VISION.md) + [`ARCHITECTURE.md`](ARCHITECTURE.md). The sections below are the original pivot reasoning, retained for context.)*

## 3. New obligations a public multi-tenant app takes on

- **UGC compliance (Guideline 1.2):** EULA, in-app **report/block**, moderation path, **age-gating (1.2.1)**.
- **Per-tenant safety of customization:** a malicious or broken layout/rule must never escalate privileges, exfiltrate another group's data, or brick the app.
- **Privacy disclosure** for any AI/third-party data sharing (Guideline 5.1.2(i)); prefer on-device.
- **Sustainability while ad-free:** CloudKit storage/traffic largely sit in each user's iCloud quota, so per-tenant cost stays near-zero — what lets "no ads, ever" survive a public release.

## 4. What carried over

Swift / SwiftUI, CloudKit + R2, push, free serverless cron, Sign in with Apple, opportunistic on-device Apple Intelligence, and the non-negotiables — privacy by default and no ads, ever. The three core jobs remain, as the default template every group reshapes.

## 5. Data-not-code (the App Store keystone, still current)

Guideline **2.5.2** forbids downloading/executing code that changes app functionality. The customization engines are **interpreters embedded in the binary that consume declarative data** — the compliant pattern, and deliberately *not* the HTML5/JS "mini-app" path (Guideline 4.7). Layouts and rules are content, not code. *(This principle carries forward unchanged into the current local-first design.)*

---

### Sources (App Store constraints, June 2026)
- [App Review Guidelines — Apple Developer](https://developer.apple.com/app-store/review/guidelines/) (2.5.2; 4.7; 1.2 / 1.2.1; 5.1.2(i))
- [App Store Guideline 1.2 — User Generated Content (BuddyBoss)](https://buddyboss.com/docs/app-store-guideline-1-2-safety-user-generated-content/)
