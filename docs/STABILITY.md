# STABILITY & SIZE — keeping it crash-proof and small

> **Status: design note (June 2026, idea phase).** Extends [`PERFORMANCE.md`](PERFORMANCE.md) with concrete techniques for **reliability** and a **small binary**. The headline: our own architecture already does most of the work — the rest is discipline.

## 0. Why we start ahead

Three choices we already made are the biggest stability + size wins there are:

- **Local-first** → the app works with no network; there's no "spinner of death," no failed-request crashes, no half-loaded screens. Offline is the normal case, not an error path. ([`VISION.md`](VISION.md) Inversion 1)
- **Data, not code** → there is **no user-supplied executable code to crash the app or bloat the binary**. Customization is tiny JSON the engine validates. ([`ARCHITECTURE.md`](ARCHITECTURE.md))
- **Assets stream from R2, not bundled** → the binary ships only the *default* theme's assets; every imported font/image/icon/motion file lives in R2 and is fetched-on-demand + cached. **App size stays small and roughly constant no matter how wildly groups customize.** ([`FEASIBILITY.md`](FEASIBILITY.md))

Everything below builds on those.

---

## 1. Stability

### 1.1 The customization engine is the main new crash surface — so harden it
A malformed view/rule/theme/motion (from sync or an imported bundle) is the most likely crash. Defenses, layered:
- **Defensive decoding.** Every document decodes with safe defaults and *tolerates unknown fields/versions* — never force-unwrap, never assume shape. Unknown/invalid → **validate-and-degrade** to the default, with an admin-visible note ([`SDUI-SPEC.md`](SDUI-SPEC.md) §7). A bad config can *never* brick a screen.
- **Total, sandboxed evaluation.** Reactions are bounded (no loops/recursion, run caps, off-main) so a rule can't hang or spin ([`RULES-SPEC.md`](RULES-SPEC.md)). The renderer treats every node as untrusted input.
- **Per-frame + resource budgets** on motion/animation so a fancy imported animation can't peg the CPU ([`PERFORMANCE.md`](PERFORMANCE.md)).

### 1.2 Safe mode / crash-loop recovery (the standout idea)
Like a browser's safe mode: **detect a crash loop on launch** (e.g. 2 crashes within N seconds) → automatically **boot with the default experience** and quarantine the group's custom config, offering "restore my customization" or "reset." A self-inflicted bad theme/layout/rule becomes a one-tap recovery, not a dead app. The **local-first history** ([`REALTIME.md`](REALTIME.md)) also gives **last-known-good rollback** for free.

### 1.3 Memory discipline = fewer kills (especially extensions)
On iOS, **out-of-memory termination** is a top "crash" cause, and **app extensions (Widgets, Live Activities, notification service) have tiny memory limits (~tens of MB)**. So:
- **Bounded everything:** LRU caches with hard ceilings, the CRDT log bounded by **snapshots + compaction**, capped in-flight work. No unbounded growth.
- **Extensions load a slim path**, never the whole engine — a precompiled default view + minimal data, not the full Creator/renderer.

### 1.4 Data integrity
- **Atomic / journaled writes** to the local store — never a half-written state; recover by replaying the log onto the last snapshot if a write is interrupted.
- **CRDT merge** removes whole classes of sync-conflict corruption; merges are deterministic and order-independent.
- **Idempotent sync + reactions** so retries (flaky network, app relaunch) never double-apply.

### 1.5 Concurrency without data races
- **Swift 6 strict concurrency** + **actors** around the store/engines: the compiler catches data races *at build time*. Single-writer model for the log; `Sendable` everywhere it crosses a boundary; structured concurrency so nothing leaks or runs after cancellation.

### 1.6 Ban the crash-prone patterns
- Lint/CI rules forbidding `!` force-unwrap, `try!`, `as!`, and unchecked array subscripts in non-test code. Prefer total functions, `guard let`, safe collection access.

### 1.7 See failures without bloat or tracking
- **MetricKit** (Apple-native, on-device) for crash + hang + hitch + memory reports — **no third-party crash SDK** (those add size *and* fight our no-tracking rule). `os_signpost` for diagnostics.

### 1.8 Test for stability, not just correctness
- **Property-based / fuzz tests** on the CRDT merge and the document decoders with pathological inputs (the golden + corrupt + newer-schema fixtures from [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md)).
- **Soak + interruption tests:** airplane mode, backgrounding mid-write, low-memory, clock changes, two devices diverging then merging.

---

## 2. Smaller app

### 2.1 The asset rule (biggest lever)
**Nothing user-customizable ships in the binary.** Default theme assets only; all imported/kit assets live in **R2**, downloaded lazily and cached, evicted under pressure. A group with 200 MB of custom fonts/graphics doesn't add a byte to anyone's app download.

### 2.2 Use Apple's free pixels
- **SF Symbols** as the default icon system — thousands of weights/scales for ~zero bytes; custom icons are user-imported (R2).
- **One variable font**, **subset to used glyphs**, instead of many static weights; user fonts via R2.
- **Vectors (PDF/SF Symbols) + HEIC**, compressed, metadata stripped.

### 2.3 Ship less code
- **Curated component registry + minimal dependencies** — no JS engine, no Flutter runtime, no analytics/ad SDKs; every dependency audited for size (and justified per [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) DoD).
- **`-Osize`, whole-module optimization, dead-code stripping, strip symbols** in release; no debug info shipped.
- **Data-driven, not code-generated.** The capability catalog is *data* (tiny); we don't generate a class per component/token. Adding customization options grows a JSON file, not the binary.

### 2.4 One shared core, not N copies
- Put the engine/model in **one shared framework/package** used by the app *and* its extensions, instead of statically linking it into every target (which multiplies size). This serves **size and stability** (the slim extension path) at once.

### 2.5 Let the App Store thin it
- **App thinning / slicing:** device-specific downloads, asset-catalog slicing, correct @2x/@3x. The user downloads only their device's slice.
- **On-Demand Resources (ODR):** tag rarely-used built-in assets (e.g. a seldom-used default template variant) to download on demand rather than at install.

### 2.6 Measure it as a gate
App-size tracked in CI against a target set at first build; a regression fails the build ([`PERFORMANCE.md`](PERFORMANCE.md) §6). Watch App Store Connect's app-size report per device.

---

## 3. The one honest trade-off

**Stability vs. size on the CRDT library.** A proven CRDT library (Automerge/Yjs-style) buys reliability but adds binary size; a hand-rolled minimal LWW-register + add-only-set is tiny but is code we must get right. Recommendation: **start minimal and hand-rolled for v1** (it covers chat/RSVP/polls/points and keeps us small), with the `SyncEngine` port making it swappable for a battle-tested library later if merge complexity grows ([`TECH-STACK.md`](TECH-STACK.md), [`CRITIQUE.md`](CRITIQUE.md) §1.2). Measure both ways before committing.

## 4. One-liner

**Because customization is data and assets stream from R2, the binary stays small and constant; because there's no user code and everything validates-and-degrades with bounded resources and a safe-mode fallback, a bad config can't crash or bloat the app.**

---

### Related
[`PERFORMANCE.md`](PERFORMANCE.md) (budgets) · [`ARCHITECTURE.md`](ARCHITECTURE.md) (ports, local-first) · [`CREATOR.md`](CREATOR.md) (fallback/recovery) · [`REALTIME.md`](REALTIME.md) (history) · [`CRITIQUE.md`](CRITIQUE.md) (CRDT log growth) · [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md) (tests/CI) · [`SECURITY.md`](SECURITY.md).
