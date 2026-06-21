# Trivselslederne (TL) — MVP & Roadmap

*Planning document for the TL app. Prepared from the group's brainstorm (Tl app.pdf). Group: "WAD?FC" — Arvind, Ruben, Fridrik, Morten, Adrian, Fredrik, Emil, Lars, Eivind (9 members).*

> **Decision (current):** the app is **iOS-only**. Arvind is the only Android user and is switching to iPhone soon, so we drop skip.dev / Android / the PWA for now. Worst case, a simple website can be added later. This simplifies everything: pure **CloudKit + Cloudflare R2**, no cross-platform backend.

---

## 1. What we're building

A private, invite-only app for **one friend group of 9**, on **iPhone**. Three jobs to do well:

1. **Plan hangouts** — where, when, who, what.
2. **Group chat** — one shared space, with read receipts.
3. **Trivselslekene** — the yearly event: scoring, leaderboard, trophies, history.

Deep iPhone integration (Widgets, Live Activities, Siri, Apple Intelligence) is a goal and — now that we're iOS-only — a first-class one, not an extra.

Because the audience is fixed at ~9 people on iPhone, we optimize for **zero hosting cost** and **simplicity**.

---

## 2. Every idea from the brainstorm, sorted

| Theme | Idea | Source | Priority |
|---|---|---|---|
| **Chat** | Group chat + **read receipts** | Fridrik / Arvind | Must (MVP) |
| **Planning** | Event planning (hvor/når/hvem/hva) | Arvind | Must (MVP) |
| **Planning** | Polls / stemme over ting | Arvind | Must (MVP) |
| **Planning** | Kalender — kryss av når du er hjemme i Sandnes | Eivind | Must (MVP) |
| **Notifications** | Push (free via CloudKit subscriptions) | — | Must (MVP) |
| **Social** | TL BeReal / daily vlog (story-style, ephemeral) | Eivind | Should (P2) |
| **Social** | Ønskelister + Secret Santa (jul/bursdag) | Arvind | Should (P2) |
| **Social** | The TL bucket list | Arvind | Should (P2) |
| **iOS-extra** | Widgets, Live Activities, Siri | Arvind | Could (P2/P3) |
| **iOS-extra** | Apple Intelligence (smart replies, summaries) | new | Could (opportunistic) |
| **iOS-extra** | Photo/video animations for Trivselslekene | Arvind | Could (P3) |
| **Gamification** | Trivsels-Leaderboard (point system) | Fridrik | Should (P3) |
| **Gamification** | Games section — trophies / achievements / records | Fridrik | Should (P3) |
| **Gamification** | Trivselslekene event module | (core purpose) | Should (P3) |
| **Money** | TL ferien 2030 — saving + planning | Arvind | Could (P4) |
| **Money** | Vipps betting / "TL polymarket" | Fridrik | Could (P4) — ⚠️ legal caveat, §6 |

---

## 3. The MVP (v1.0)

The smallest version that's genuinely useful on day one, cheap and safe to run.

**In scope:**

- **Login** — Sign in with Apple; membership is the CloudKit shared zone (the 9 of you). No public sign-up.
- **Group chat** — one channel, text + images, **read receipts**, push on new message. (Near-real-time: messages appear within seconds — see §6 on why this isn't live co-editing.)
- **Events** — create a hangout (title, place, date/time, description), RSVP (going / maybe / no), comments.
- **Polls** — ask a question, options, everyone votes, live results.
- **"Hjemme i Sandnes" calendar** — each person marks the days they're home; everyone sees an overlap view.
- **Push notifications** — new message, new event, new poll, event reminder (free, no server).

**Out of MVP** (deferred): BeReal/vlogs, wishlists/Secret Santa, bucket list, leaderboard/achievements, Trivselslekene module, savings, betting, widgets/Live Activities/Siri, Apple Intelligence.

---

## 4. Hosting — free and secure, no server

### The short answer

**Apple CloudKit** for app data + **Cloudflare R2** for photo/video files. Both have permanent free tiers that are enormous for 9 people, neither needs a server you run, and push notifications are free. The only paid item anywhere is the **Apple Developer membership (~$99/yr)** you need to ship to iPhones at all.

### CloudKit (app backend)

- **Free tier, generous for 9:** 10 GB asset storage, 100 MB database **+250 MB per user**, 2 GB/day transfer, 40 requests/sec. ([Apple — CloudKit](https://developer.apple.com/icloud/cloudkit/))
- **Almost no setup:** tied to the iCloud accounts you already have. No separate project, no card on file.
- **The CKShare "shared zone" *is* your group** — one person owns it, the other 8 join. That replaces any allowlist / security-rule layer.
- **Free push with no server:** CloudKit **subscriptions** (CKQuerySubscription) send a notification automatically when records change. ([Hacking with Swift](https://www.hackingwithswift.com/read/33/8/delivering-notifications-with-cloudkit-push-messages-ckquerysubscription))
- Native fit with Widgets, Live Activities, Siri, Sign in with Apple, and Apple Intelligence.

### Cloudflare R2 (media files)

- **10 GB storage, zero egress fees, permanently free.** ([R2 pricing](https://developers.cloudflare.com/r2/pricing/)) Purpose-built for photo/video — the one thing that strains CloudKit's quota and 2 GB/day transfer cap.
- Store the *file* in R2 (uploaded via a tiny free Cloudflare Worker); keep the *metadata* (who/when/caption/expiry) in CloudKit.

### Do we need a server to "push on"?

Mostly no. Push notifications need **no server** (CloudKit does it). For the few scheduled jobs — pick "who's it" today, delete expired clips, sign R2 uploads — use a **free serverless cron**: Cloudflare Workers free tier or Val Town. There's no always-on server to rent or maintain.

### What G Suite legacy gives you

Not a backend, but handy: a **custom domain** for the R2 bucket / any future website, and **Drive** for design files and periodic data backups.

### Security

- Membership = CKShare participants; outsiders can't reach the zone.
- Sign in with Apple — no passwords to store.
- CloudKit enforces permissions server-side, independent of the app.
- Encrypted in transit and at rest by default.

### If the group ever needs cross-platform again

Revisit **Firebase (Spark, free)** — natively iOS+Android, doesn't pause when idle. Avoid Supabase (its free projects pause after a week of inactivity). Not needed while the group is all-iPhone.

---

## 5. Roadmap

Loose, hobby-pace phasing. Each phase ships to TestFlight (which also keeps you clear of App Store UGC review — see §6).

### Phase 0 — Foundations
Register the Apple Developer Program (unlocks CloudKit + TestFlight). Set up a CloudKit container + a CKShare shared zone for the group, Sign in with Apple, and a free Cloudflare account (R2 bucket reserved for Phase 2 media). Agree a simple design system (colors, the "WAD?FC" logo, app icon). Outcome: empty iPhone app that logs in the group and nothing else.

### Phase 1 — MVP (the core loop)
Group chat + read receipts → Events + RSVP → Polls → "Hjemme i Sandnes" calendar → Push. Outcome: the group actually uses it to plan hangouts. **This is the real launch.**

### Phase 2 — Social & iOS flavor
TL daily vlog / BeReal (story-style ephemeral, §7), Ønskelister + Secret Santa, the TL bucket list. Start the iOS-extras: a **Home Screen widget** (next event / who's home), a **Live Activity** for an in-progress hangout, a **Siri** shortcut to mark "I'm home in Sandnes," and opportunistic **Apple Intelligence** (smart replies, chat summaries) gated to capable devices. Outcome: fun to open daily, not just when planning.

### Phase 3 — Trivselslekene & gamification
Point system + leaderboard, trophies / achievements / records, and a dedicated **Trivselslekene module** (events, live scoring, history, the turning-around player animations). Outcome: the app owns your signature tradition.

### Phase 4 — Money features (with care)
**TL ferien 2030**: a shared savings goal + tracker + planning board (no money held by the app). **Vipps "TL polymarket"**: see §6 — launch as a **points-only / non-monetary** prediction game first.

A reasonable order: Phase 0–1 first (get it in hands), then 2 and 3 by group excitement, with 4 last (most legal/financial nuance).

---

## 6. Risks & decisions to make

**Real-time co-editing (the "Google Docs" question).** CloudKit syncs with a few seconds' latency — perfect for chat, polls, events, read receipts — but it is **not** live simultaneous text editing. True Google-Docs-style co-editing needs CRDT + a realtime channel, which CloudKit doesn't provide. Recommendation: **skip it for MVP** — nothing in the feature list truly needs it. If you later want a shared *live* note, flag it and we'll design that one feature on a realtime add-on. *Decision: confirm we don't need live co-editing in v1.*

**Vipps betting.** Norway has a strict state gambling monopoly. Informal betting *between friends* is generally allowed **as long as it isn't run as a business** — no operator taking a cut. ([ICLG Norway 2026](https://iclg.com/practice-areas/gambling-laws-and-regulations/norway/), [Gambling in Norway — Wikipedia](https://en.wikipedia.org/wiki/Gambling_in_Norway)) An app that **holds or routes money, or takes a fee, risks being treated as a commercial gambling operator.** Safe path: **non-monetary points/prediction game**. If you ever attach real money, keep settlement strictly peer-to-peer in Vipps with the app taking nothing, and get advice first. General info, not legal advice.

**App Store UGC rules (Guideline 1.2).** Chat + vlogs = user-generated content, so a *public* release needs terms/EULA, report + block, and content-removal tools. ([Apple guidelines](https://developer.apple.com/app-store/review/guidelines/)) For 9 friends, **stay TestFlight-only** (up to 100 testers) to sidestep this entirely; add report/block/terms only if you ever go public.

**Sync conflicts & offline.** Two people editing the same event, or posting offline — use last-write-wins via CloudKit change tokens, and test airplane mode.

**Group ownership / bus factor.** The CKShare zone has one owner — if that account lapses, the group's data is at risk. Document who owns it and keep periodic exports to Drive.

*(Upkeep — membership renewal, push certs, updates — Arvind is handling.)*

---

## 7. The daily vlog / BeReal — "a Snapchat story that lasts longer"

This works exactly as you want: it's an **ephemeral shared feed with a lifespan you choose** (e.g. 3–7 days instead of 24h — story-style, but longer).

The one constraint: there's **no API for apps to post into a native iCloud Shared Album** ([Apple forum](https://developer.apple.com/forums/thread/717066)). So instead of Apple's Shared Albums, you build your **own shared album in the CloudKit shared zone** (which you're doing anyway), and let each person **save their own clip to their personal Photos** for permanence (that *is* allowed).

**Storage is a retention problem, not a quota problem.** Because clips expire, steady-state stays tiny regardless of quality — so you can **keep compression light** (your preference) without blowing the budget. Math: even everyone-every-day kept 48h ≈ 9 × 2 × ~15 MB ≈ **270 MB at any time**. Only keeping clips *forever* would explode it.

Setup:

- **Files on R2** (10 GB, zero egress), metadata + expiry in CloudKit.
- **Light compression** is fine — HEVC at near-original quality; just cap absurd lengths.
- **Lifespan is a setting** — start at a few days; tune to taste.
- **Save-your-own** to Photos for anything someone wants to keep.

Design decisions:

- **Who's "it" today** needs no server: every client hashes (date + a shared group secret) to independently pick the same person; handle a no-show with a grace window / reroll.
- Define "the day" in **one timezone (Europe/Oslo)**.
- **Consent/deletion:** anyone can delete their own clip; agree a norm on screenshots/saving.

---

## 8. Other capabilities & notes

- **Read receipts** — easy and native: each viewer writes a "seen" marker record; everyone reads them. In the MVP.
- **Apple Intelligence (opportunistic).** The **Foundation Models framework** gives on-device, free, private AI — smart replies, thread/plan summaries, "suggest a hangout" from chat. Needs iPhone 15 Pro / 16-family + iOS 26 ([Apple docs](https://developer.apple.com/documentation/FoundationModels)); gate gracefully for older phones, never a hard dependency. Bonus: the App Store Small Business Program can unlock larger Private Cloud Compute models at no API cost.
- **Push** — free, no server (CloudKit subscriptions). Live Activity / widget *remote* updates need a tiny push sender later (free serverless).
- **Backups** — periodic CloudKit export to Drive so the group's history is safe.

---

## 9. Recommended next steps

1. Register the Apple Developer Program (~$99/yr) — unlocks CloudKit and TestFlight.
2. Create a CloudKit container + CKShare shared zone for the group; wire up Sign in with Apple.
3. Create a free Cloudflare account + an R2 bucket (used from Phase 2 for vlog media).
4. Scaffold the SwiftUI app; lock in the design system and WAD?FC branding. Distribute **TestFlight-only** for v1.
5. Build Phase 1 (MVP), ship to TestFlight, and let real usage decide whether Phase 2 (social) or Phase 3 (Trivselslekene) comes next.

---

### Tech stack at a glance
Swift / SwiftUI (iOS-only) · Sign in with Apple · CloudKit (container + CKShare; free) · CloudKit subscriptions for push · Cloudflare R2 for vlog media (free, zero egress) · free serverless cron (Cloudflare Workers / Val Town) for scheduled jobs · Apple Intelligence / Foundation Models (opportunistic) · Vipps (peer-to-peer only, later) · custom domain + Drive from G Suite legacy. *Cross-platform fallback if ever needed: Firebase (Spark).*

*Sources: [Apple — CloudKit](https://developer.apple.com/icloud/cloudkit/) · [CloudKit subscriptions](https://www.hackingwithswift.com/read/33/8/delivering-notifications-with-cloudkit-push-messages-ckquerysubscription) · [Cloudflare R2 pricing](https://developers.cloudflare.com/r2/pricing/) · [Foundation Models framework](https://developer.apple.com/documentation/FoundationModels) · [iCloud Shared Album API limitation](https://developer.apple.com/forums/thread/717066) · [Apple App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) · [Firebase pricing](https://firebase.google.com/pricing) · [ICLG Gambling Norway 2026](https://iclg.com/practice-areas/gambling-laws-and-regulations/norway/) · [Gambling in Norway — Wikipedia](https://en.wikipedia.org/wiki/Gambling_in_Norway)*
