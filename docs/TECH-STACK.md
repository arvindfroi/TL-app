# TECH-STACK — Swift/iOS-only, and how to stay free to improve later

> **Status: reasoning (June 2026, idea phase).** Companion to [`ARCHITECTURE.md`](ARCHITECTURE.md), [`PERFORMANCE.md`](PERFORMANCE.md), [`PRIOR-ART.md`](PRIOR-ART.md).
>
> ✅ **Decided ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D3): iOS-only; Android is *not pursued*.** The cross-platform/Skip discussion below is **considered, not pursued** — kept for context. We still keep the **thin seam** (core/model import no SwiftUI/CloudKit), but only for **testability + the CRDT Plan-B swap**, not portability.

## 1. Short answer

**Swift / SwiftUI, iOS-only.** The differentiators — a *fast* SDUI renderer, deep iOS integration (App Intents/Siri, Widgets), on-device **Foundation Models** — are best done native; a cross-platform runtime makes them slower and harder. The crew is all-iPhone, so Android buys nothing in v1.

The one nuance worth keeping: the thing that would *actually* lock us to Apple isn't the language — it's **CloudKit**. So we keep a **thin seam** (below) for cleanliness and to make the CRDT Plan-B swap easy, not because we plan to leave iOS.

## 2. Why iOS-only native is right

- **The hard part is best done native.** A fast renderer + deep Apple integration + on-device AI are exactly what a cross-platform runtime degrades. Native is what hits the [`PERFORMANCE.md`](PERFORMANCE.md) budgets.
- **One platform, one team, one truth.** Two-person hobby build; a second platform multiplies surface area for no v1 benefit.
- **Everyone's on iPhone.** Android isn't a launch requirement — or a pursued one ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) D3).
- **It keeps us slim.** No JS engine, no Flutter runtime, no abstraction tax on the hot path.

## 3. The thin seam (cleanliness, not portability)

Even iOS-only, it's worth keeping the brain decoupled from Apple specifics:

1. **Core (pure Swift, no SwiftUI, no CloudKit):** data model, the view compiler, the reaction evaluator, validation, catalogs, RBAC. Framework-agnostic, unit-testable.
2. **Adapters (protocols + Apple impls):** `Store`, `SyncEngine`, `BlobStore`, `Push`, with CloudKit/R2/APNs implementations.
3. **Shell (SwiftUI):** the renderer, screens, the group switcher, navigation.

Why bother if we're iOS-only? Because it makes the code **testable**, and makes the **CRDT Plan-B swap** (CloudKit-as-source-of-truth) a one-adapter change instead of a rewrite ([`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L3). It is **not** an investment in Android.

## 4. Cross-platform (considered, not pursued)

For the record: **Skip** (skip.dev) can transpile SwiftUI→Jetpack Compose, and Swift 6.3 runs on Android — but SwiftUI doesn't run natively there, the Android SDK is still developer-preview, and **CloudKit won't run on Android at all** (it'd need a different `SyncEngine` adapter). Given the crew is all-iPhone, **we don't pursue this.** Alternatives (React Native, Flutter, Kotlin Multiplatform) trade away the native speed + Apple-AI integration that are the whole point. If Android ever became real, the thin seam means it's "a second renderer + a non-Apple sync adapter," but that's explicitly out of scope.

## 4b. "Can't it just be React and CSS?"

Keep their mental models, not their runtimes.

- **CSS — yes, in spirit.** Our theme tokens + declarative style props **are** a CSS-like system: semantic properties cascading onto components, with the layered override model in [`CREATOR.md`](CREATOR.md) §8. It compiles to **native SwiftUI** instead of a browser.
- **React — like it structurally, not the runtime.** Layout documents are a tree of components with props + children — the same declarative/composable model as JSX. We just don't **execute** user-supplied JavaScript.

**Why not actual React/JS-in-a-webview:** (1) App Store 2.5.2/4.7 — executing brought-in JS is downloaded code; a webview "mini-app" hits 4.7 and can't bridge native APIs. (2) A webview is slower, non-native, and can't cleanly reach App Intents/Foundation Models/Widgets/the local-first store. (3) Declarative documents survive better and are easier for a future AI to author. **Net:** React-like components + CSS-like styling as **declarative data**, rendered natively.

## 5. How to "function like we want now, improve later" (evolvability playbook)

1. **Walking skeleton first.** The thinnest end-to-end slice — create group → invite → one themed chat — on the *real* architecture (core + adapters + shell). See [`DATA-MODEL.md`](DATA-MODEL.md) §10.
2. **Contracts before implementations.** Freeze the *document schemas* and *adapter protocols* early; improve the implementations behind them without breaking callers.
3. **Catalogs are the extension points.** New capability = a new registry/data-source/action entry, not a new code path.
4. **Modular packages (SPM)** with enforced boundaries: a package importing SwiftUI into Core is a lint failure.
5. **Hybrid rendering, on purpose.** Hot/complex screens native; SDUI where customization matters ([`PRIOR-ART.md`](PRIOR-ART.md) §1).
6. **Don't go fully generic too early.** Domain-specific components (EventCard, PollCard), generalize only on real demand.
7. **Versioning + migrations from day one.** Every document/record carries a schema version.
8. **Feature flags / dark launches.**
9. **Event/append-only where it helps** (the op log) — syncs cleanly, gives an audit trail.

## 6. One-line policy

**iOS-only Swift/SwiftUI because the differentiators are native; the brain is pure portable Swift behind a thin seam (for testability + the CRDT Plan-B swap), and CloudKit is one adapter — but Android is not pursued.**
