# PERFORMANCE — how "extremely customizable" stays slim, fast, and smooth

> **Status: design principle + budgets (proposed, June 2026).** Companion to [`PIVOT.md`](PIVOT.md), [`SDUI-SPEC.md`](SDUI-SPEC.md), [`RULES-SPEC.md`](RULES-SPEC.md). This is a **standing constraint on every PR**, not a phase. If a feature can't meet the budgets in §6, it isn't done.

## 1. The core tension (state it honestly)

Deep customization (editable layouts + a rules engine) is the thing most likely to make the app **slow, janky, and bloated** — a runtime that re-parses JSON, does dictionary lookups, and rebuilds the whole view tree on every keystroke is the classic SDUI failure. "Slim, fast, smooth" is therefore not a vibe; it's the **architecture that makes customization free at runtime.** The whole strategy below is one idea applied everywhere: **do the expensive work once, off the hot path, and let the steady state be plain typed Swift.**

## 2. Compile, don't interpret (the single most important rule)

A layout document or rule set is **parsed and validated exactly once**, at load/save time, into a **typed, resolved in-memory representation** (a "compiled layout / compiled ruleset" — an IR of Swift structs/enums). The render and evaluation **hot paths walk typed values, never JSON.**

- **Token resolution happens at compile time** — `"md"` → `12pt`, color roles → concrete colors for the active theme. The renderer never resolves tokens per frame.
- **The compiled form is cached** (in memory, and to disk keyed by document hash + schema + theme), so a layout compiles once per change, not once per appearance.
- **Default templates ship precompiled** — they are authored as native Swift (or compiled at build time), so the common path (a group that hasn't customized) pays **zero parsing cost at runtime**. Custom layouts are the only ones that ever compile on device, and only once.

Net effect: a customized screen and a hardcoded screen have **the same steady-state cost.**

## 3. Render only what changed

- **Stable identity:** every compiled node carries a stable id so SwiftUI diffing re-renders just the subtree that changed, not the screen.
- **Granular bindings:** a binding observes only its data slice (`chat.recent`), so a new message re-renders the message list — not the event card, not the header. No "god object" that invalidates everything.
- **Lazy by default:** collections use `LazyVStack`/`List`; only visible rows are built. Off-screen sections aren't materialized.
- **No work in `body`:** the compiled IR means `body` is pure layout assembly — no parsing, no formatting, no allocation churn. Expensive derivations (date formatting, overlap computation) are memoized.

## 4. The rules engine must be cheap and invisible

- **Off the main thread, always.** Evaluation runs on a background queue; only the resulting UI-affecting action hops back to main.
- **Indexed by trigger:** a data change consults only rules registered for that trigger type — not a linear scan of all rules. Conditions short-circuit (`all`/`any`).
- **Coalesced & debounced:** bursts of changes (e.g. nine RSVPs landing via sync) collapse into one evaluation pass.
- **Bounded:** per-rule and per-group run caps (from [`RULES-SPEC.md`](RULES-SPEC.md)) cap worst-case cost; dedup/claim means the work happens on one device, not nine.
- **Scheduled rules** wake via free serverless cron, not a polling loop on device.

## 5. Slim — keep the binary and the footprint small

- **Curated component registry, not a kitchen sink.** Customization is *composition + theming within a fixed, lean set of native components* — not an unbounded component zoo. This is what lets us be both highly customizable *and* small. Add a component only when a real need appears.
- **Minimal dependencies.** Native SwiftUI + at most a couple of tiny, audited packages (e.g. the variable-font helper). **No JavaScript engine, no heavyweight cross-platform runtime, no analytics/ad SDKs** (also a non-negotiable). Every dependency is a size + startup + risk cost.
- **Asset discipline:** SF Symbols first; subset variable fonts to the axes/glyphs used; compress and lazy-load images; thumbnails from R2, full media on demand.
- **Data slimness:** CloudKit **delta sync** via change tokens — fetch only what changed and only what the visible screen binds to; paginate; cache. Layout/rule documents are tiny JSON. Vlogs stay ephemeral so storage doesn't creep.
- **Build hygiene:** strip unused code/assets, dead-strip symbols, watch app-size regressions in CI. (More in [`STABILITY.md`](STABILITY.md) §2.)

## 6. Performance budgets (the definition of "done")

Concrete, measurable, enforced. Numbers are targets to validate on the oldest supported device, not aspirations:

- **Cold start → interactive:** ≤ ~400 ms for the default-template path (no on-device document parsing on this path at all).
- **Custom-layout first paint:** ≤ ~100 ms compile for a typical screen, then cached (≈0 thereafter).
- **Scrolling:** sustained **60 fps (120 fps on ProMotion)** — frame budget 16.6 ms / 8.3 ms; **zero JSON parsing or layout compilation on the main thread, ever.**
- **Interaction latency:** tap → visible response < 100 ms.
- **Rule evaluation:** a triggered pass completes < ~16 ms of main-thread time (ideally 0 — it's off-main); no user-perceptible stall.
- **Memory:** steady-state ceiling per screen; no unbounded caches (compiled-layout cache is LRU-bounded).
- **App size:** set a target download size at Phase 1 and treat regressions as bugs.

## 7. Smooth — motion that never fights the frame

- **Spring-based, interruptible animations**; the variable-font axis animation from [`DESIGN.md`](../DESIGN.md) interpolates on a capped cadence, not per-frame recompute.
- **Honor Reduce Motion** (tone down/stop) — also an accessibility floor enforced by the renderer.
- **Optimistic UI:** local writes apply instantly; CloudKit sync reconciles in the background (last-write-wins). The user never waits on the network to feel a tap land.
- **Skeletons over spinners** for bound data that's still loading; never block the screen on a fetch.

## 7b. The fast & smooth playbook (concrete iOS/SwiftUI)

**One rule: the main thread is sacred** — it does layout + render only; everything else runs off it. Every frame has ~**8 ms on 120 Hz ProMotion** (16 ms at 60 Hz); exceed it and you drop a frame.

- **Fine-grained observation:** use the `@Observable` macro (iOS 17+), not `ObservableObject`/`@Published`, so only views reading a changed property invalidate — a new message doesn't redraw the screen.
- **Help diffing:** stable `Identifiable` IDs on rows; cheap value-type views; **avoid `AnyView`** (defeats diffing); don't overuse `GeometryReader`.
- **Lazy + cheap rows:** `LazyVStack`/`List`, known sizes to skip layout passes, trivial row `body` (no formatting/allocation).
- **Heavy drawing → GPU:** `.drawingGroup()` / `Canvas`/Metal for intense effects (Trivselslekene, imported motion).
- **Image pipeline (top scroll-hitch cause):** thumbnails from R2, **downsample to display size off-main** (ImageIO), cache memory+disk; never decode full-res on the main thread.
- **Instant feel:** local-first writes apply optimistically (never wait on network/disk); cold start shows a **skeleton** while the store hydrates and images decode off-main; cache `DateFormatter`s and memoize derived values.
- **Concurrency priority:** interaction at `userInitiated`, sync/compaction/rule-eval at `background`, so a sync never competes with a scroll.
- **Honor** Reduce Motion + Low Power Mode (simpler effects, 60 Hz).

## 8. Measure methodically (don't guess)

- **Instruments + `os_signpost`** around compile, render, sync, and rule evaluation.
- **Automated performance tests** (XCTest metrics) for scroll, cold start, and layout-compile time, run in CI against the budgets in §6 — a regression fails the build.
- **Golden layout/rule documents** (small, large, pathological) used as fixtures for both correctness (snapshot tests) and performance.
- **Frame-drop / hitch tracking** (MetricKit) in TestFlight builds so real-device jank surfaces early.

## 9. The one-line policy

**Customization is data that compiles once into typed Swift; the running app is always just native SwiftUI over a lean component set, measured against hard budgets.** If a change can't hold that line, it gets redesigned — not shipped slow.
