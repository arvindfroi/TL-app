# REALTIME — does local-first still give us real-time chat (and more)?

> **Status: design note (June 2026, idea phase).** Answers a load-bearing worry about [`VISION.md`](VISION.md): if the group's state is local-first/CRDT, do we lose real-time chat? Short answer: **no — local-first makes chat *better*, not slower.**

## 1. The key distinction: "instant local" vs "propagation speed"

"Real-time" is two separate things:

1. **Local responsiveness** — how fast *your* message appears for *you* when you send. Local-first makes this **instant, always**, even offline: you write to your own device, the UI updates immediately, no network round-trip. Strictly better than cloud-first.
2. **Propagation latency** — how fast it reaches *others' phones*. This depends on the **transport**, not on local-first. Local-first just guarantees that whenever a change arrives — in 200 ms or 2 minutes — it **merges correctly**.

So local-first removes the lag you feel and turns "real-time" into a pure transport question we can dial up over time.

## 2. CRDTs are the *ideal* substrate for group chat

- **Concurrent senders merge cleanly.** Three people typing at once, out-of-order arrival, someone offline who reconnects — the log converges to the same conversation on every phone, no duplicates, no "message failed to send."
- **Offline just works.** Write on the plane; it slots into place on landing.
- **No central referee.** Ordering is handled by the data structure (timestamps + stable IDs), not a server.

## 3. Two lanes of data (the important design idea)

| Lane | Examples | Where it lives | Delivery |
|---|---|---|---|
| **Durable state** | messages, events, RSVPs, polls + tallies, points, read receipts, reactions | the **CRDT log** (source of truth, synced, kept) | reliable, merges, survives offline |
| **Ephemeral signals** | typing indicators, "online now" presence, "looking at this" | a **fast, lossy channel** — never persisted | best-effort; a dropped packet doesn't matter |

Typing/presence don't belong in durable history. A separate lightweight channel keeps the log clean *and* makes presence feel instant.

## 4. The transport spectrum (how fast "real-time" is)

- **Tier 0 — CloudKit subscriptions (free, no server, already planned).** Push on record change; ~1–3s end-to-end. For 9 friends this reads as "basically instant." Local echo means *your* side is always instant.
- **Tier 1 — APNs nudges + delta pull.** Trims latency, powers notifications. Still effectively serverless.
- **Tier 2 — a thin real-time channel (later, optional).** For **sub-second** delivery + typing/presence, a lightweight pub/sub/WebSocket relay shuttles ephemeral signals. The CRDT log stays source of truth; the relay is *only* a faster pipe, so it can be lossy and stateless.

Ship Tier 0 (+1) for v1; add Tier 2 only if we want true typing/presence — a clean upgrade, not a rebuild.

## 5. "And more" — the live features this enables

- **Live poll results** as votes land (durable, tiny — fine on Tier 0).
- **Reactions / read receipts** near-instant (durable, tiny).
- **Live Trivselslekene scoring** — scores are durable objects; leaderboard updates as entered.
- **Typing & presence** — ephemeral lane; want Tier 2 to feel crisp.
- **Live Activities / widgets** reflecting an in-progress hangout.
- **Voice/video** is not something we build — hand off to FaceTime / a call link.

## 6. The one honest caveat (and the $0-hosting tension)

The single thing local-first + CloudKit doesn't hand us free is **true sub-second presence/typing** — it wants a persistent connection, which brushes our **"no paid always-on server"** rule. Options, cheapest first: (1) push presence/typing **through CloudKit/APNs** at lower fidelity — serverless, "good enough"; (2) a **free/cheap managed real-time tier** for *only* ephemeral signals; (3) decide it **isn't worth it for v1** and ship without. None change the architecture.

## 7. App Store & privacy notes

- **Still data-not-code** — a relay only moves messages/signals (2.5.2 untouched).
- **Privacy:** durable content syncs through the group's encrypted store; a relay (if added) carries minimal ephemeral signals, ideally with E2E-encrypted message payloads (relay sees ciphertext).
- **No ads, no tracking** — unchanged.

## 8. Bottom line

Local-first **upgrades** chat: instant on your side, offline-proof, conflict-free on merge. "Real-time across devices" is a transport dial — **near-real-time (seconds) for free on day one**, **sub-second + typing/presence** as a small optional later add-on. We lose nothing by going local-first; we gain speed, resilience, and a clean path to richer live features.

---

### Related
[`VISION.md`](VISION.md) (Inversion 1) · [`ARCHITECTURE.md`](ARCHITECTURE.md) · [`PERFORMANCE.md`](PERFORMANCE.md) · [`TECH-STACK.md`](TECH-STACK.md). On CRDTs/local-first: [Local-first software — Ink & Switch](https://www.inkandswitch.com/local-first.html), [Automerge](https://automerge.org/).
