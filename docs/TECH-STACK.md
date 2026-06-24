# TECH-STACK — is Swift-only the move, and how to stay free to improve later

> **Status: reasoning / not yet decided (June 2026, idea phase).** Answers two questions: *should we be Swift-only?* and *how do we architect so the app works the way we want now but stays cheap to change later?* Companion to [`ARCHITECTURE.md`](ARCHITECTURE.md), [`PERFORMANCE.md`](PERFORMANCE.md), [`PRIOR-ART.md`](PRIOR-ART.md). Nothing here is locked; it's here to be argued with.

## 1. Short answer

**Yes — Swift / SwiftUI only, for now.** But the real insight is that *the language was never the lock-in.* Swift now runs on Android (official Swift 6.3 Android support; Skip transpiles SwiftUI→Jetpack Compose). The thing that actually ties us to Apple is **CloudKit**. So the smart move isn't to hedge on the language — it's to **keep Swift and isolate the two things that would be expensive to change later: the UI framework and the backend.** Do that and "improve later / go cross-platform later" stays a small, deliberate project instead of a rewrite.

## 2. Why Swift-only is right *now*

- **The hard part is best done native.** Our differentiators — a *fast* SDUI renderer, deep iOS integration (App Intents/Siri, Widgets, Live Activities), and on-device **Foundation Models** — are exactly what a cross-platform runtime makes slower and harder. Going native is what lets us hit the [`PERFORMANCE.md`](PERFORMANCE.md) budgets.
- **One platform, one team, one truth.** We're a two-person hobby build. A second platform now multiplies surface area before we've proven the product with our own group.
- **Everyone's on iPhone.** The audience is the crew first; Android is a "maybe later," not a launch requirement.
- **It keeps us slim.** No JS engine, no Flutter runtime, no abstraction tax on the hot path.

## 3. The trap to avoid: confusing "Swift-only" with "Apple-welded"

If we sprinkle `CloudKit` and `SwiftUI` types through every layer, we get an app that's Swift-only *and* permanently iOS-only, because the **backend (CloudKit) is Apple-only and is the true Android blocker.** The fix is architectural, not linguistic:

**Portable core + thin native shell (ports & adapters).** Three rings:

1. **Core (pure Swift, no SwiftUI, no CloudKit):** the data model, the SDUI document/compiler, the rules DSL + evaluator, validation, the capability catalogs, RBAC logic. This is ~the whole brain of the app, and it's framework-agnostic, unit-testable, and portable.
2. **Adapters (protocols + Apple implementations):** persistence/sync (`Store`, `SyncEngine`), push, media, AI provider, all defined as **protocols** in Core, with CloudKit/R2/APNs/FoundationModels implementations in an outer ring.
3. **Shell (SwiftUI):** the renderer turns the *compiled* layout into SwiftUI; screens, editor, navigation.

Consequences that buy us freedom:
- Swapping CloudKit later (for Android, or if quotas/economics change) means writing one new **adapter**, not touching Core.
- A future Android build via **Skip** reuses Core + ports a second renderer/adapters — the document format is already platform-neutral ([`SDUI-SPEC.md`](SDUI-SPEC.md) §11).
- Core has no UI, so it's trivially testable and AI-legible (the schemas live here).

## 4. Reality check on the cross-platform future (so we plan honestly)

- **Skip** (skip.dev) is now open-source with official Swift-on-Android support — genuinely promising, and the lowest-friction "SwiftUI on Android" path. **But** the Swift Android SDK is still developer-preview and the DX isn't production-grade yet. So: treat Skip as a **future option we keep open, not a dependency we assume.** Validate with a small spike *before* committing (it's already listed as a future spike in [`EXECUTION-PLAN.md`](EXECUTION-PLAN.md)).
- **The honest blocker remains the backend.** Even with Skip, CloudKit won't run on Android. If Android ever becomes real, we either (a) put a portable sync backend behind our `SyncEngine` port (Firebase, or a local-first engine — see [`PRIOR-ART.md`](PRIOR-ART.md) §5), or (b) keep CloudKit for iOS and add a second adapter for Android. Designing the port now is what makes either choice cheap.
- **Alternatives we are *not* taking (and why):** React Native/Expo (JS runtime tax + OTA-UI sits awkwardly with App Store rules + weaker native-AI story), Flutter (non-native feel, bridges needed for App Intents/Foundation Models), Kotlin Multiplatform (shares logic but not SwiftUI; we'd still write UI twice). Each trades away the native speed and Apple-AI integration that are the whole point right now.

## 4b. "Can't it just be React and CSS?"

Reasonable question — and the answer is **keep their mental models, not their runtimes.**

- **CSS — yes, in spirit.** Our theme tokens + declarative style props **are** a CSS-like system: semantic properties cascading onto components, with the layered override model in [`CREATOR.md`](CREATOR.md) §8 (essentially CSS specificity). The advanced editor can even use CSS-ish syntax. It just compiles to **native SwiftUI** instead of a browser.
- **React — like it structurally, not the runtime.** Layout documents are a tree of components with props + children — the same declarative/composable model as JSX. A React dev feels at home. We just don't **execute** user-supplied JavaScript.

**Why not actual React/JS-in-a-webview:**
1. **App Store 2.5.2 / 4.7.** Executing brought-in JS = downloaded code (2.5.2); a webview "mini-app" falls under Guideline 4.7, which forbids bridging native APIs (4.7.2) without approval. Riskiest footing for review.
2. **It kills the differentiators.** A webview is slower, non-native (fights [`PERFORMANCE.md`](PERFORMANCE.md)), and can't cleanly reach App Intents/Siri, Foundation Models, Widgets, or the local-first store — the deep-iPhone integration that's the whole point.
3. **Portability + AI prefer data.** Declarative documents survive to a future Android renderer and are far easier for the AI to author than code.

**Net:** users get **React-like components + CSS-like styling as declarative data**, rendered natively. A contained `WKWebView` for one exotic surface is possible later, but stays off the core path for the three reasons above.

## 5. How to "function like we want now, improve later" (evolvability playbook)

1. **Walking skeleton first.** Build the thinnest end-to-end slice — create group → invite → join → one themed chat screen — on the *real* architecture (Core + ports + shell), not a throwaway. Everything after is thickening, not rework.
2. **Contracts before implementations.** Freeze the *document schemas* and the *port protocols* early; let the implementations behind them be dumb at first and improve internally without breaking callers.
3. **Catalogs are the extension points.** New capability = a new registry/data-source/action entry, not a new code path. Growth is additive.
4. **Modular packages (SPM)** with enforced boundaries: `Core`, `SDUIEngine`, `RulesEngine`, `Persistence`, `DesignTokens`, `AppShell`. A package that imports SwiftUI into Core is a lint failure.
5. **Hybrid rendering, on purpose.** Not every screen needs to be SDUI. Render the **hot/complex screens natively** and reserve SDUI for where customization actually matters. (Airbnb/Netflix pattern; antidote to Spotify's over-abstraction — [`PRIOR-ART.md`](PRIOR-ART.md) §1.)
6. **Don't go fully generic too early.** Ship the default template as **domain-specific components** (EventCard, PollCard…), not a soup of primitives. Generalize a component only when a group actually wants to bend it.
7. **Versioning + migrations from day one.** Every document and record type carries a schema version; readers tolerate newer/older.
8. **Feature flags / dark launches.** Land engines behind flags, enable per group.
9. **Event/append-only where it helps.** Chat, rule-execution logs, points — append-only logs sync cleanly, give an audit trail, and feed the AI.

## 6. One-line policy

**Swift/SwiftUI native now, because the differentiators are native; but the brain of the app is pure portable Swift behind protocols, and CloudKit is just one adapter — so we can get faster, swap the backend, or add Android later without a rewrite.**
