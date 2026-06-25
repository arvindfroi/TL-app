# DATA-MODEL — the one object graph (canonical spec)

> **Status: spec (June 2026, idea phase).** The foundation everything depends on, written concretely so views, reactions, sync, and AI all build on the same ground. Companion to [`VISION.md`](VISION.md) (Inversion 2), [`ARCHITECTURE.md`](ARCHITECTURE.md), [`REALTIME.md`](REALTIME.md), [`SECURITY.md`](SECURITY.md), [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md). This is the doc to nail before any code.

## 1. Principles

- **One graph, domain-typed.** Everything in a group is an **object** with a shared envelope and a typed body (`Event`, `Poll`, `Message`, `Tradition`, `Theme`, `View`, `Reaction`…). Unification is at the *storage/sync/validation* layer; the **types stay specific** (no generic "Node" soup — the HubFramework trap, [`PRIOR-ART.md`](PRIOR-ART.md)).
- **Local-first + CRDT.** Each device holds the source of truth; edits are **operations** that merge conflict-free. Cloud is transport ([`REALTIME.md`](REALTIME.md)). *(Plan B if this proves too hard: §6.)*
- **Facets, not separate engines — *semantically unified, physically separated*.** A screen (*view*), an automation (*reaction*), and a look (*theme/motion*) are **objects of their own types** that *reference a target*, stored/synced/validated exactly like domain objects. "One model" = one envelope, one store, one sync, one validator — **not** literally embedding UI inside an Event, and **not** one mega-renderer. In code, the **View renderer, Reaction evaluator, and Theme resolver are independent, well-bounded modules** behind explicit interfaces; only the *plumbing* (envelope/storage/sync/validation) is shared. This is the deliberate guard against the Spotify-HubFramework over-abstraction trap ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L2).
- **Data, not code; validate-and-degrade.** Every object is validated against its type schema on apply; unknown/invalid → quarantine or fall back, never crash ([`STABILITY.md`](STABILITY.md)).

## 2. The object envelope (every object has this)

```jsonc
{
  "id": "01J9Z3K8...",        // ULID — sortable, globally unique, offline-generatable
  "type": "event",            // discriminator → which typed body + schema
  "schema": 1,                // per-type schema version (forward-compatible)
  "group": "01J9...",         // the group (zone) this belongs to
  "author": "did:...",        // who created it (stable member id)
  "createdAt": "HLC...",      // Hybrid Logical Clock (see §3)
  "updatedAt": "HLC...",      // HLC of last applied op
  "deleted": false,           // tombstone (compacted later)
  "fields": { /* typed body, per §4 */ },
  "facetOf": null             // for view/reaction/look objects: the target id (else null)
}
```

State is never written directly; it's the **fold of operations** (§5). The envelope above is the *materialized* form.

## 3. Identity & time

- **IDs: ULID** (or UUIDv7) — generated offline on-device, sortable by creation time, collision-free across devices. No server round-trip to mint an id.
- **Ordering: Hybrid Logical Clocks (HLC)** — a `(wallClock, counter, deviceId)` triple that gives a total order consistent with causality without trusting device clocks. HLC stamps every operation; ties break by `deviceId`.
- **Members & multi-group:** one **Sign in with Apple** identity per person, used as `author`. A person can belong to **many groups** → one **`Member` record per (person, group)**, carrying a **per-group profile** (display name/avatar that can differ per group). The app holds a **current active group** and renders only that zone; the user **switches groups like switching accounts on Instagram** ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D2). **No data crosses groups** — each group is a private island; only lightweight notification badges may surface across groups.

## 4. Object types (v1 vocabulary)

**Domain:** `Group`, `Member`, `Message`, `Event`, `RSVP`, `Poll`, `Vote`, `AvailabilityDay` ("hjemme i Sandnes"), `PointsEntry`, `Tradition` (recurring ritual with history).
**Customization (facet objects):** `View` (a layout/screen, [`SDUI-SPEC.md`](SDUI-SPEC.md)), `Reaction` (a rule, [`RULES-SPEC.md`](RULES-SPEC.md)), `Theme` (tokens + motion), `AssetRef` (pointer to a blob in R2 by content hash).

Each type has a **schema** (allowed fields + their CRDT kind, §6) that the validator and the AI's capability manifest both consume.

## 5. The operation log (how state is built)

- Every change is an **operation**: `{ opId(ULID), hlc, author, group, target(id), type, op-body }` — e.g. `set field`, `add to set`, `remove from set`, `increment`, `create`, `tombstone`.
- Each device keeps an **append-only log** per group; **materialized state = fold(log)**. New ops arrive via sync and merge by HLC.
- **Growth is bounded** by periodic **snapshots + compaction**: fold the log into a checkpoint, drop superseded ops/tombstones, keep recent history for undo/time-travel ([`CRITIQUE.md`](CRITIQUE.md) §1.2). "Unlimited history" in practice = rich recent + milestone snapshots.

## 6. Conflict resolution — CRDT strategy per field kind

This is the part the data model lives or dies on. Each field declares a **kind**; merge is defined per kind, so "two people edit the same object offline" has a precise, non-corrupting answer:

| Field kind | Used for | Merge rule |
|---|---|---|
| **LWW-Register** | scalars: title, date, RSVP status, a theme token value | last write wins by **HLC** (tie: deviceId). The losing edit is retained in history, not lost forever. |
| **OR-Set** (add-wins) | sets: poll options, tags, members, availability days | concurrent adds all survive; a remove only cancels adds it observed (tombstone). |
| **PN-Counter** | points/tallies | per-member increments summed — concurrent points never clobber. |
| **Append log** | chat messages | messages are immutable, add-only; the conversation is the ordered set of `Message` objects (HLC order). Edits = a new `editedBody` LWW field. |
| **Ordered list** | a view's child order | v1: an explicit `order` array as LWW; **later: fractional indexing** (Figma-style keys) if concurrent reorders need to both survive. |
| **Tombstone** | deletion | mark `deleted`; compaction removes later. |

**Worked example — two people edit the same `Reaction` offline:** A changes its `schedule` field, B changes its `enabled` field → different fields, both apply, no conflict. If both change `enabled`, **LWW by HLC** picks one; the rule is never corrupted, both devices converge to the same value, and history shows both edits. For a 9-person high-trust group this is the right trade (we are explicitly **not** building Google-Docs co-editing — [`REALTIME.md`](REALTIME.md)).

**Merge explainability (a real UX risk).** Local-first systems are notoriously hard to debug when "my edit vanished." Mitigation: keep merges **legible** — every field shows *who last set it and when* (author + HLC), the history scrubber shows the superseded value, and an LWW-overwrite can surface a quiet "Ruben's later change won" note rather than a silent disappearance. If a field's merge can't be made explainable, that's a signal to choose a different CRDT kind for it.

**Plan B (named up front).** CRDTs have real composition/tombstone/explainability cost ([`CRITIQUE.md`](CRITIQUE.md) §1.2, [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3). **Spike L decides:** if full CRDT-over-CloudKit proves uncontrollable, fall back to **CloudKit as source of truth + optimistic local updates** — we lose pure offline-merge but keep the instant feel and most of the local-first benefit. Local-first is the goal, not a religion; this is the documented escape hatch.

## 7. Permissions in the model

Every operation carries its `author`. On apply (locally *and* server-side via CloudKit), the engine checks: *may this author write this object/field in this group, given their role?* ([`SECURITY.md`](SECURITY.md) §2). RBAC is light inside a group ([`VISION.md`](VISION.md) Inversion 4) but enforced in the data layer, never just the UI. Reads are permission-filtered so a `View` binding can't surface data the member can't see.

## 8. Mapping to storage

- **Operations/objects → CloudKit records** in the group's zone (fields: envelope + op-body + HLC + author). CloudKit subscriptions push new ops to members ([`REALTIME.md`](REALTIME.md)).
- **Blobs** (images, fonts, motion files) → **R2**, addressed by **content hash**; objects hold an `AssetRef` (hash + size + type), never the bytes. Dedup + cache for free.
- **Optional E2EE:** encrypt op-bodies/blobs client-side so CloudKit/R2 hold ciphertext ([`SECURITY.md`](SECURITY.md) §3).
- A **thin seam** ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D3): the model knows nothing about CloudKit — it talks to a `Store`/`SyncEngine` protocol; CloudKit is one adapter.

## 9. Schema versioning & migration

- Every object/op carries `schema`. Readers **tolerate unknown fields and newer versions** (decode with defaults), so an older app never breaks on a newer object — it degrades that object to a safe form.
- Migrations are **forward-only** transforms registered per type; applied lazily on read or in a snapshot pass.

## 10. The v1 minimal subset (the walking skeleton)

Don't build all types at once. The first end-to-end slice is just: **`Group` + `Member` + `Message`**, over the **local-first op log**, rendered by **one hardcoded view**, synced via CloudKit transport — proving create-group → invite → offline-message → merge. (This is exactly the local-only-first prototype, the way to de-risk everything; see [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) Spikes L & A and [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3.) Add `Event`/`Poll`/`Theme`/`Reaction` as new *types*, not new engines.

## 11. Examples

```jsonc
// Event
{ "id":"…","type":"event","schema":1,"group":"g1","author":"m_arvind",
  "createdAt":"HLC","updatedAt":"HLC","deleted":false,
  "fields":{ "title":{"k":"lww","v":"Ski trip"},
             "startsAt":{"k":"lww","v":"2030-02-14T17:00+01:00"},
             "venue":{"k":"lww","v":null},
             "rsvps":{"k":"orset","v":{"m_ruben":"going","m_emil":"maybe"}} } }

// Reaction (a "command block")
{ "id":"…","type":"reaction","schema":1,"group":"g1","author":"m_arvind","facetOf":null,
  "fields":{ "enabled":{"k":"lww","v":true},
             "trigger":{"k":"lww","v":"event.created"},
             "when":{"k":"lww","v":{"all":[{"field":"event.startsAt.dayOfWeek","op":"in","value":["Sat","Sun"]}]}},
             "do":{"k":"lww","v":[{"action":"createPoll","with":{"question":"Where?"}}]} } }
```

## 12. Open questions

- **Ordered lists:** LWW `order` array (simple, occasional reorder loss) vs **fractional indexing** (robust concurrent reorder, more code) — decide when layout editing gets real.
- **Message edits/deletes:** LWW `editedBody` vs immutable + tombstone — pick one early.
- **OR-Set GC:** when can add-tombstones be compacted safely?
- **HLC vs vector clocks:** HLC is lighter and enough for LWW/causality here; revisit only if a feature needs true concurrency detection.
- **CRDT vs Plan B:** Spike L decides (§6).
- **Per-group key rotation** on member removal (forward secrecy) — interacts with the op log ([`SECURITY.md`](SECURITY.md) §2).

---

### Related
[`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) (L2/L3, D2/D3) · [`VISION.md`](VISION.md) (Inversion 2) · [`ARCHITECTURE.md`](ARCHITECTURE.md) · [`SDUI-SPEC.md`](SDUI-SPEC.md) (View) · [`RULES-SPEC.md`](RULES-SPEC.md) (Reaction) · [`REALTIME.md`](REALTIME.md) (sync) · [`SECURITY.md`](SECURITY.md) (permissions, E2EE) · [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) (the prototype that proves this).
