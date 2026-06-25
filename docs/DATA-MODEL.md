# DATA-MODEL — the one object graph (canonical spec)

> **Status: spec (June 2026, idea phase).** The foundation everything depends on, written concretely so views, reactions, sync, and AI all build on the same ground. Companion to [`VISION.md`](VISION.md) (Inversion 2), [`ARCHITECTURE.md`](ARCHITECTURE.md), [`REALTIME.md`](REALTIME.md), [`SECURITY.md`](SECURITY.md). This is the doc to nail before any code.

## 1. Principles

- **One graph, domain-typed.** Everything in a group is an **object** with a shared envelope and a typed body (`Event`, `Poll`, `Message`, `Tradition`, `Theme`, `View`, `Reaction`…). Unification is at the *storage/sync/validation* layer; the **types stay specific** (no generic "Node" soup — the HubFramework trap, [`PRIOR-ART.md`](PRIOR-ART.md)).
- **Local-first + CRDT.** Each device holds the source of truth; edits are **operations** that merge conflict-free. Cloud is transport ([`REALTIME.md`](REALTIME.md)).
- **Facets, not separate engines.** A screen (*view*), an automation (*reaction*), and a look (*theme/motion*) are **objects of their own types** that *reference a target*, stored/synced/validated exactly like domain objects. "One model" = one envelope, one store, one sync, one validator — **not** literally embedding UI inside an Event.
- **Data, not code; validate-and-degrade.** Every object is validated against its type schema on apply; unknown/invalid → quarantine or fall back, never crash ([`STABILITY.md`](STABILITY.md)).

## 2. The object envelope (every object has this)

```jsonc
{
  "id": "01J9Z3K8...",        // ULID — sortable, globally unique, offline-generatable
  "type": "event",            // discriminator → which typed body + schema
  "schema": 1,                // per-type schema version (forward-compatible)
  "group": "01J9...",         // the group (CKShare zone) this belongs to
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
- **Ordering: Hybrid Logical Clocks (HLC)** — a `(wallClock, counter, deviceId)` triple that gives a total order consistent with causality without trusting device clocks. HLC stamps every operation; ties break by `deviceId`. This is how we order concurrent edits deterministically on every device.
- **Members:** a stable member id per person (derived from Sign in with Apple, mapped once); used as `author` and in permissions.

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
| **Append log** | chat messages | messages are immutable, add-only; the conversation is the ordered set of `Message` objects (HLC order). Edits = a new `editedBody` LWW field on the message. |
| **Ordered list** | a view's child order | v1: an explicit `order` array as LWW; **later: fractional indexing** (Figma-style keys) if concurrent reorders need to both survive. |
| **Tombstone** | deletion | mark `deleted`; compaction removes later. Deletes lose to concurrent edits only per the field rule. |

**Worked example — two people edit the same `Reaction` offline:** A changes its `schedule` field, B changes its `enabled` field → different fields, both apply, no conflict. If both change `enabled`, **LWW by HLC** picks one; the rule is never corrupted, both devices converge to the same value, and history shows both edits. For a 9-person high-trust group this is the right trade (we are explicitly **not** building Google-Docs co-editing — [`REALTIME.md`](REALTIME.md)).

## 7. Permissions in the model

Every operation carries its `author`. On apply (locally *and* server-side via CloudKit), the engine checks: *may this author write this object/field in this group, given their role?* ([`SECURITY.md`](SECURITY.md) §2). RBAC is light inside a group ([`VISION.md`](VISION.md) Inversion 4) but enforced in the data layer, never just the UI. Reads are permission-filtered so a `View` binding can't surface data the member can't see.

## 8. Mapping to storage

- **Operations/objects → CloudKit records** in the group's CKShare zone (fields: envelope + op-body + HLC + author). CloudKit subscriptions push new ops to members ([`REALTIME.md`](REALTIME.md)).
- **Blobs** (images, fonts, motion files) → **R2**, addressed by **content hash**; objects hold an `AssetRef` (hash + size + type), never the bytes. Dedup + cache for free.
- **Optional E2EE:** encrypt op-bodies/blobs client-side so CloudKit/R2 hold ciphertext ([`SECURITY.md`](SECURITY.md) §3).
- The `Store`/`SyncEngine` **ports** ([`TECH-STACK.md`](TECH-STACK.md)) mean this mapping is one adapter; the model itself knows nothing about CloudKit.

## 9. Schema versioning & migration

- Every object/op carries `schema`. Readers **tolerate unknown fields and newer versions** (decode with defaults), so an older app never breaks on a newer object — it degrades that object to a safe form.
- Migrations are **forward-only** transforms registered per type; applied lazily on read or in a snapshot pass.

## 10. The v1 minimal subset (the walking skeleton)

Don't build all types at once. The first end-to-end slice is just: **`Group` + `Member` + `Message`**, over the **local-first op log**, rendered by **one hardcoded view**, synced via CloudKit transport — proving create-group → invite → offline-message → merge. (This is exactly the local-only-first prototype, the way to de-risk everything; see [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) Spikes L & A.) Add `Event`/`Poll`/`Theme`/`Reaction` as new *types*, not new engines.

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
- **Per-group key rotation** on member removal (forward secrecy) — interacts with the op log ([`SECURITY.md`](SECURITY.md) §2).

---

### Related
[`VISION.md`](VISION.md) (Inversion 2) · [`ARCHITECTURE.md`](ARCHITECTURE.md) · [`SDUI-SPEC.md`](SDUI-SPEC.md) (View) · [`RULES-SPEC.md`](RULES-SPEC.md) (Reaction) · [`REALTIME.md`](REALTIME.md) (sync) · [`SECURITY.md`](SECURITY.md) (permissions, E2EE) · [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) (the prototype that proves this).
