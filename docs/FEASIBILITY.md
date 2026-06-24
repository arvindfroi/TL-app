# FEASIBILITY — the three pillars, and how they ship free on CloudKit + R2

> **Status: feasibility check (June 2026, idea phase).** Three concrete references for what the app should feel like, mapped to our architecture and to the **free** stack (CloudKit + Cloudflare R2, no server we run). Verdict: **yes, the core is achievable for free — with two small asterisks, both with free workarounds.**

## The three pillars → references → our design

| Pillar | Reference | What it means | Specced in |
|---|---|---|---|
| **Customizable UI** | **Cocoon Shell** | Theme builder, every color editable with **live preview**, swappable **asset packs**, mix-and-match, **export/share** your setup, a community store | [`CREATOR.md`](CREATOR.md) + [`SDUI-SPEC.md`](SDUI-SPEC.md) |
| **Rules** | **Minecraft command blocks** | Declarative trigger→action automations; a command-palette feel; shareable like datapacks | [`RULES-SPEC.md`](RULES-SPEC.md) + [`CREATOR.md`](CREATOR.md) §5 |
| **Real-time chat** | **Snapchat** | Instant-feeling messaging, presence/typing, ephemeral media | [`REALTIME.md`](REALTIME.md) |

**Cocoon Shell basically *is* a proof-of-concept of pillar 1** — a customizable "shell" with a theme builder + live preview + asset packs + a community store (**Silk Pod**) + exportable theme bundles. That's our Creator, validated by a shipping product.

## Pillar 1 — Cocoon Shell-style UI customization (free ✅)

A group's look is a small set of **documents** — theme tokens (every color/roundness/spacing/motion), the layout/views, and references to assets. Editing is the [`CREATOR.md`](CREATOR.md) Design surface with **live preview**, exactly like Cocoon's Theme Builder.

- **Theme + layout documents** → **CloudKit** (tiny JSON; sync to everyone). Effectively free.
- **Assets** (wallpapers, icon packs, sounds, hero images/GIF/video, **fonts, motion files**) → **Cloudflare R2** (10 GB free, **zero egress**) — keeps heavy files off CloudKit's quota.
- **Export/share a theme bundle** → a manifest (JSON) + assets; importing pulls and validates them. Free for sharing within and between your own groups.

**Asterisk (small):** a **public discovery store** like Silk Pod needs a shared, moderated catalog — but it can be **third-party** (the user is fine with that), since the bundle format is open. **Free workaround now:** share bundles peer-to-peer (link/file). You get 100% of creation + sharing free; only the public storefront is deferred/third-party.

## Pillar 2 — Minecraft command-block rules (free ✅)

Automations are declarative **rule documents** (trigger → condition → action), authored via node canvas / command palette ([`CREATOR.md`](CREATOR.md) §5), evaluated **on-device**, deduplicated ([`RULES-SPEC.md`](RULES-SPEC.md) §7).

- **Rule documents** → CloudKit (tiny). Free.
- **Evaluation** → on each phone, background thread. No server. Free.
- **Scheduled rules** → a **free serverless cron** (Workers / Val Town) nudges. Still $0.
- **Sharing rules as "datapacks"** → same bundle/export path.

No asterisk — comfortably free, and App-Store-safe because rules are **data, not code** (2.5.2).

## Pillar 3 — Snapchat-style real-time chat (free ✅, one honest caveat)

| Snapchat feel | How we do it free | Verdict |
|---|---|---|
| **Your message appears instantly when you send** | **Local-first**: write to your own device, UI updates immediately | ✅ instant, even offline |
| **Friends get it a beat later** | **CloudKit push** (CKQuerySubscription via APNs) → ~1–2s | ✅ "feels instant", free, no server |
| **Photos/videos/snaps** | Media to **R2** (zero egress), a tiny reference syncs via CloudKit | ✅ free, exactly R2's job |
| **Ephemeral / disappearing** | Expiry on message/media; R2 object deleted on expiry | ✅ free, and a storage win |
| **Reactions, read receipts** | Tiny durable records, merge via the CRDT log | ✅ free |
| **"Ruben is typing…" + presence** | **Ephemeral lane** — fast, lossy, never saved | ⚠️ the caveat ↓ |

**The one honest caveat:** truly **sub-second typing/presence** wants a persistent real-time connection CloudKit doesn't provide, which brushes the "no always-on server" line. It does **not** affect message delivery — chat still feels instant. **Free options:** (1) low-fidelity presence over CloudKit/APNs; (2) a free real-time tier used *only* for ephemeral signals; (3) ship v1 without typing/presence. Detail in [`REALTIME.md`](REALTIME.md).

## The free-tier budget (why this costs ~$0)

| Need | Service | Free allowance | Fits us? |
|---|---|---|---|
| App data, chat, configs, rules, sync, push | **CloudKit** | 10 GB asset + 100 MB DB **+250 MB/user**, 2 GB/day transfer; free push | Enormous for a friend group |
| Photos / video / theme assets | **Cloudflare R2** | 10 GB storage, **zero egress** | Purpose-built; ephemerality keeps it tiny |
| Scheduled rule wake-ups | **Workers / Val Town** | Free serverless cron | Trivial load |
| Identity | **Sign in with Apple** | Free | ✅ |
| **Only recurring cost** | **Apple Developer Program** | ~$99/yr | The one real bill |

The two things that can push past free, both deferrable: a **public theme store** (can be third-party) and **crisp always-on presence**. Neither blocks v1.

## Verdict

**It's possible, and the core is free.** Cocoon-style customization, command-block rules, and Snapchat-feel chat all map onto CloudKit (sync/data/push) + R2 (media/assets) + on-device logic, no server we run, only the $99/yr Apple membership. The two asterisks — a *public* store (third-party is fine) and *sub-second* presence — are small, deferrable, and have free paths. These three pillars *are* the heart ([`CRITIQUE.md`](CRITIQUE.md) §6).

---

### Sources & related
- [Cocoon Shell — Theme Builder](https://cocoon-shell.com/wiki/theme-builder/) · [Customization](https://cocoon-shell.com/wiki/customization/) · [Themes / Silk Pod store](https://cocoon-shell.com/wiki/themes/)
- Internal: [`CREATOR.md`](CREATOR.md) · [`SDUI-SPEC.md`](SDUI-SPEC.md) · [`RULES-SPEC.md`](RULES-SPEC.md) · [`REALTIME.md`](REALTIME.md) · [`CRITIQUE.md`](CRITIQUE.md) · [`ARCHITECTURE.md`](ARCHITECTURE.md)
