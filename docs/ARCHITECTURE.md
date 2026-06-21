# Architecture (brief)

The full reasoning lives in [`MVP-and-Roadmap.md`](MVP-and-Roadmap.md) §4. This is the short version for orientation.

## Principles
- **iOS-only.** Everyone is on iPhone, so we go all-in on Apple's stack — no cross-platform layer.
- **No server to run.** Use managed free tiers; add tiny serverless functions only when a feature truly needs scheduled logic.
- **Closed group.** Membership is the CloudKit shared zone, not an open sign-up.

## Components

| Concern | Choice | Why |
|---|---|---|
| App data (chat, events, polls, availability, read receipts) | **CloudKit** (CKShare shared zone) | Free, tied to existing iCloud accounts, no backend; the share *is* the group |
| Push notifications | **CloudKit subscriptions** (CKQuerySubscription) | Free, no server — fires on record changes |
| Photo / video files (vlogs) | **Cloudflare R2** | 10 GB free, **zero egress**; metadata stays in CloudKit |
| Scheduled jobs (daily "who's it", expiry, R2 upload signing) | **Cloudflare Workers / Val Town** free tier | Free serverless cron; no always-on server |
| Auth | **Sign in with Apple** | No passwords stored |
| On-device AI (optional) | **Foundation Models framework** | Free, private, on-device; gated to capable devices |

## Data flow notes
- **Sync** is near-real-time (seconds), not live co-editing. Fine for chat/polls/events. True Google-Docs simultaneous editing is out of scope (would need CRDT + realtime channel).
- **Conflicts**: last-write-wins via CloudKit change tokens.
- **Vlogs**: ephemeral by design (story-style, configurable lifespan). Files in R2, metadata + expiry in CloudKit. Steady-state storage stays small because clips expire.
- **Backups**: periodic CloudKit export to Drive.

## Cross-platform fallback (not active)
If the group ever needs Android again: **Firebase (Spark, free)** — natively iOS+Android, doesn't pause when idle. Avoid Supabase (free projects pause after a week idle).
