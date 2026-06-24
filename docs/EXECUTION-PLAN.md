# EXECUTION-PLAN — making the build seamless and swift

> **Status: execution playbook (proposed, June 2026).** Turns the pivot plan into something a small team can build without stalling. Companion to [`PIVOT.md`](PIVOT.md) §8 (roadmap) and [`PERFORMANCE.md`](PERFORMANCE.md). Audience: contributors (currently Arvind + Ruben).

## 1. Principle: de-risk before you commit

The two new engines are the only genuinely novel/risky parts. Everything else (chat, events, polls on CloudKit) is well-trodden. So we **spike the risky things first**, behind throwaway prototypes, and only then commit to the phase plan. This is what keeps execution swift — we don't discover the hard problem in month three.

## 2. De-risking spikes (do these in Phase 0/1, timeboxed, disposable)

- **Spike A — SDUI render performance.** Hand-compile one real screen (the chat or event screen) to the IR from [`PERFORMANCE.md`](PERFORMANCE.md) §2 and prove 60/120 fps scroll + the cold-start budget on the oldest target device. *Kills the biggest risk: that customization is inherently janky.*
- **Spike B — Multi-CKShare tenancy.** Prove one user participating in **two** groups (two CKShare zones), data isolated, RBAC records read correctly, invite accept flow works. *Validates the core architecture shift.*
- **Spike C — Rules dedup.** Prove the claim-record gives exactly-once execution when 3 simulated devices see the same trigger. *Validates the automation engine's correctness model.*
- **Spike D — Customization sync + fallback.** Prove a corrupt/newer layout document degrades to the default without a crash, and that an edit on device A appears on device B in seconds. *Validates the safety story.*

Each spike has one yes/no question and a short writeup of the answer. Code is thrown away; the **learning** feeds the real implementation.

## 3. Critical path (dependencies, not just phases)

Build order is dictated by what unblocks the most:

```
Apple Dev acct (#1)
   └─> CloudKit container (#2 rework: zone-per-group)
          ├─> Sign in with Apple (#3)
          ├─> Group + RBAC + Invites  ← NEW, the Phase-0 keystone
          │       └─> Theme tokens ──> Core loop (chat/events/polls/cal/push)  [Phase 1 = launch]
          │                                 └─> SDUI engine [Phase 2]
          │                                        └─> Rules engine [Phase 3]
          └─> R2 bucket (#5, lazy until media)
Public release + moderation + template gallery [Phase 4]  ← gated on Phase 2+3
Modules: vlog, widgets, Trivselslekene, money [Phase 5]
```

**Keystone:** Group + RBAC + Invites. Nothing multi-tenant works until it exists, and it's the issue most changed by the pivot. Build it first, build it right.

## 4. Definition of Ready (before an issue is picked up)

An issue is *ready* when it has: a clear outcome, its phase + area labels, named dependencies (blocked-by), an explicit **data model touchpoint** (which CloudKit record types/fields), a **permission note** (which roles can do it), and — if it renders UI or runs automation — a **performance note** referencing the relevant budget. No issue starts without these; that's what prevents mid-build thrash.

## 5. Definition of Done (before a PR merges)

Builds clean; unit tests pass; **snapshot tests** for any SDUI component; **performance tests green against [`PERFORMANCE.md`](PERFORMANCE.md) §6 budgets**; permission checks enforced in the **data layer** (not just UI); accessibility floors hold (Dynamic Type, contrast, VoiceOver labels, Reduce Motion); no secrets committed; docs/issue checklist updated; **no new dependency without justification**.

## 6. Testing strategy (tuned to the engines)

- **Unit:** data model, RBAC decisions, rule condition/action logic, token resolution.
- **Snapshot:** render golden layout documents → compare to reference images (light + dark, Dynamic Type sizes). Catches "a layout change broke a screen."
- **Golden documents:** a fixture set (minimal / large / pathological / newer-schema / corrupt) exercised by both the renderer and the rules evaluator for correctness *and* performance.
- **Performance:** XCTest metrics for cold start, scroll fps, layout-compile time, rule-eval time — in CI, failing on regression.
- **Integration:** the multi-CKShare flows (invite, join, two-group isolation) on real iCloud accounts; airplane-mode/offline reconciliation.

## 7. CI gates (cheap automation that prevents stalls)

- Build + unit/snapshot/perf tests on every PR.
- **Doc-link check** (the repo's docs cross-reference heavily — broken links fail).
- **App-size check** (flag binary-size regressions).
- Lint/format. Secret scanning (the `.gitignore` patterns already exist).
- The existing **`update-status.yml`** keeps `STATUS.md` current — re-point it at the new phase labels.

## 8. Branching & collaboration (small team, keep it light)

Trunk-ish: short-lived branches named `phase-N/slug`, small PRs, link the issue (`Closes #N`), one review where practical. With two active contributors, optimize for **small, frequently-merged PRs** over long-lived branches — fewer conflicts, faster feedback. The pivot docs land as one reviewable PR so Ruben sees the whole shape at once.

## 9. Tech-debt guardrails (so "fast now" doesn't mean "slow later")

- The **component registry is curated** — adding one is a deliberate decision, reviewed for need + performance, not a free-for-all.
- The **action allow-list** for rules grows by demand, each entry reviewed for abuse + cost.
- **No feature that needs a paid always-on server** (keeps the $0-leaning, ad-free model intact).
- **Customization never bypasses the permission layer or the zone boundary** — reviewed on every PR that touches the engines.
- Revisit budgets at each phase boundary; a budget you keep missing is a design smell, not a number to relax.

## 10. What "swift execution" looks like, concretely

Phase 0 ends with: a person can create a group, invite Ruben, he joins, both see an empty themed shell — on TestFlight. Phase 1 ends with the crew actually planning hangouts in it. Every phase after that ships a usable increment to TestFlight, and the public release (Phase 4) only happens once the customization engines and moderation tooling are proven. Each step is small, measured, and reversible.
