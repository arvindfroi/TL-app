# AI-READINESS — designing the app so AI can help configure and script it

> **Status: design principle + forward plan (proposed, June 2026).** AI features are **down the line**, but the *content model* must be AI-ready **now** so it isn't a costly retrofit. Companion to [`SDUI-SPEC.md`](SDUI-SPEC.md), [`RULES-SPEC.md`](RULES-SPEC.md), [`ARCHITECTURE.md`](ARCHITECTURE.md), [`PERFORMANCE.md`](PERFORMANCE.md). Scope-limited by what App Store review allows (§6).

## 1. The thesis: AI-friendly by construction

We already made the decision that unlocks AI almost for free: **layouts, themes, configs, and rules/scripts are declarative documents with documented schemas, a fixed component registry, and a closed action allow-list** (data, not code — see [`DECISIONS.md`](DECISIONS.md) ADR-005). That same structure is what an AI needs to read and write the app.

So the policy is simple: **the customization schema *is* the AI's API.** An AI assistant is just **another client of the exact pipeline the in-app editor uses** — it proposes a document (or a diff), the engine validates it, the user previews and approves, and only then is it applied. The AI **never executes anything**; it emits data that the embedded, sandboxed engines validate like any other. This is simultaneously the safety model and the App Store-compliance model.

> One line: **the editor and the AI are two clients of one validated, data-only API. AI proposes; the engine validates; the user approves; nothing runs as code.**

## 2. The interaction loop (the safety keystone)

Every AI action follows the same path, regardless of backend:

```
user intent ("make our chat screen more compact")
   → AI proposes a document/diff (layout | theme | rule | config)
   → engine VALIDATES it (schema, registry/allow-list, RBAC, accessibility, perf)
   → DIFF PREVIEW shown to the user (before/after, plain-language summary)
   → user APPROVES (RBAC-gated: only roles allowed to edit that surface)
   → applied + VERSIONED (one-tap rollback, audit log)
```

Properties that make this safe and reviewable:
- **Re-validated no matter the source.** AI output gets the same `validate-and-degrade` treatment as a hand-edited document — an invalid or out-of-catalog proposal is rejected, never applied blindly.
- **Bounded surface.** The AI can only express things the catalogs already allow (existing components, data sources, triggers, actions). It cannot invent a new capability, reach another group's data, or escalate a role.
- **Human-in-the-loop by default.** Changes are proposals until a permitted user approves. (A future "auto-apply trusted rule tweaks" mode would still be RBAC-gated and logged.)
- **Reversible.** Versioning + audit log mean any AI change is inspectable and one tap from undo.

## 3. Make the content AI-legible (do this now)

For an AI to be reliable, the app must describe itself in machine-readable form. These are cheap to build now and are also what powers the editor and validation:

- **Published schemas** (JSON Schema or equivalent) for each document type: layout, theme tokens, rule, group config. Versioned alongside the engine schema versions.
- **Capability manifest** — the machine-readable catalogs the engines already need, exported as the AI's grounding context: the **component registry** (with allowed props + bound data shapes), the **data-source catalog**, and the **trigger / condition / action catalog**. The AI is grounded *only* on this, so it proposes catalog-valid documents.
- **Stable IDs + semantic names + doc-strings.** Every component, token, source, trigger, and action carries a stable id, a human/semantic name, and a one-line description. This is what lets the AI map "move events to the top" onto the right node, and what makes its proposals legible to the user.
- **Self-describing errors.** Validation failures return structured, human-readable reasons so an AI can repair its own proposal in a second pass.

## 4. The AI connector abstraction (pluggable backends)

A single internal interface — call it the **assistant provider** — so backends are swappable and the rest of the app doesn't care which is active:

- **On-device (default): Apple Foundation Models.** Free, private, no data leaves the device, no third-party sharing → **no extra consent burden** (§6). Gated to capable devices, graceful fallback when unavailable. This is the preferred path for config/script help.
- **External connectors (opt-in): cloud LLMs.** A pluggable connector (e.g. a hosted model, or "bring-your-own-key") for heavier reasoning. **Off by default**, enabled per group/user with explicit consent and disclosure (§6).
- **Structured output / function-calling** maps the model onto the **capability manifest**: the model emits a document or a sequence of catalog operations (`createRule`, `editLayoutSection`, `setThemeToken`, …), never free-form code. Output is re-validated by the engine regardless of provider.
- **Grounding + guardrails:** the provider gets the schemas + the current document + the relevant data slice (minimized, permission-filtered) — never another group's data, never more than the task needs.

This abstraction means: ship on-device first; add external connectors later without touching the engines.

## 5. Apple Intelligence & Siri (via App Intents)

As of WWDC 2026, **App Intents is the only way Siri and Apple Intelligence reach into a third-party app** (SiriKit is deprecated). This is the same "expose capabilities as a structured catalog" pattern as our engines, so the two reinforce each other:

- **Expose core actions and content as App Intents** conforming to **assistant schemas** — Siri/Apple Intelligence can then invoke them by natural language, no fixed phrases, and content lands in the **Spotlight semantic index** with attribution back to the app.
- **Examples:** "Mark me home in Sandnes," "RSVP yes to Friday," "What's the next hangout?" — each an App Intent. The same intents are the **internal tool surface** an in-app AI connector can call, so we define them once and reuse them for Siri, Apple Intelligence, Spotlight, Shortcuts, widgets, and the assistant.
- **View Annotations** (map on-screen views to entities) later enable "make *this* screen simpler" style on-screen references.
- **Always gated + graceful:** Apple Intelligence features are limited to capable devices and **never a hard dependency**; the app is fully usable without them.

## 6. App Store guardrails (the hard limits — "as much as review allows")

- **2.5.2 — data, not code (unchanged, central).** AI assistance produces **declarative documents** (a layout, a theme, a rules-DSL "script"), interpreted by the embedded engines. The model never downloads or executes code, and "write me a script" means "emit a validated rules document," not runnable Swift/JS. Keeps us clear of 2.5.2 and of the 4.7 mini-app regime.
- **5.1.2(i) — third-party AI consent (updated Nov 2025).** Before any personal/group data is sent to a **third-party AI**, the app must **disclose which third party and why, and get explicit in-app consent** (a visible prompt — a privacy-policy link is not enough). So: external connectors are **off by default, opt-in, clearly scoped**; **on-device Foundation Models need none of this** because nothing is shared. Surface a clear indicator when an external model is in use.
- **Privacy & tenancy.** Prefer on-device; minimize and permission-filter anything sent to any model; **never send another group's data** (the zone boundary holds for AI too); no training on user content without explicit consent.
- **Device gating.** Apple Intelligence / Foundation Models require capable hardware + recent iOS; gate gracefully, never block core features.
- **Non-negotiables still apply.** No ads, privacy-by-default — AI features must not become a data-harvesting or ad vector.

## 7. What to build now vs. later

**Now (so AI isn't a retrofit) — folds into the engine work, near-zero extra cost:**
- Documented, versioned **schemas** for every customization document.
- A **capability manifest** (registry + data sources + triggers/actions) with stable IDs, semantic names, doc-strings, and structured validation errors.
- One **programmatic create/edit/validate API** that the in-app editor uses — so an AI client can reuse it verbatim.
- Wrap core actions as **App Intents** as they're built (also gives Siri/Shortcuts/widgets for free).

**Later (down the line, its own phase):**
- The **assistant-provider** interface + on-device Foundation Models integration (config/script help, "explain why this rule didn't fire," theme suggestions).
- **External connector(s)** with the 5.1.2(i) consent flow.
- **Siri / Apple Intelligence** assistant-schema adoption + View Annotations.

## 8. Concrete use cases this enables

- *"Make our chat screen more compact and move events to the top."* → AI emits a **layout diff**; preview; approve.
- *"Set up a rule: remind everyone the night before any event."* → AI emits a **rules document**; validated against the action allow-list; approve.
- *"Why didn't my weekend-poll rule fire last Saturday?"* → AI reads the **execution/audit log** and explains.
- *"Give us a cozy ski-trip theme."* → AI proposes **theme tokens** within the accessibility floors.
- *"Hey Siri, mark me home in Sandnes."* → an **App Intent**, no in-app AI needed.

## 9. Where this lands in the plan

Cross-cutting, mostly **Phase 5+**, but with **two cheap obligations pulled into Phases 1–3**: (a) keep every customization surface schema-first with a programmatic, validated edit API, and (b) wrap core actions as App Intents as they ship. Those two make the later AI work additive instead of a rebuild. Tracked as a standing principle (this doc) plus backlog items; see [`BACKLOG.md`](BACKLOG.md) and [`DECISIONS.md`](DECISIONS.md) ADR-009.

---

### Sources (verified June 2026)
- [Integrating actions with Siri and Apple Intelligence — Apple Developer (App Intents)](https://developer.apple.com/documentation/AppIntents/Integrating-actions-with-siri-and-apple-intelligence)
- [Making actions and content discoverable by Apple Intelligence — Apple Developer](https://developer.apple.com/documentation/appintents/integrating-your-app-with-siri-and-apple-intelligence)
- [Apple's new App Review Guidelines clamp down on apps sharing personal data with third-party AI — TechCrunch (Nov 2025)](https://techcrunch.com/2025/11/13/apples-new-app-review-guidelines-clamp-down-on-apps-sharing-personal-data-with-third-party-ai/)
- [Guideline 5.1.2(i) — the AI data-sharing rule (DEV)](https://dev.to/arshtechpro/apples-guideline-512i-the-ai-data-sharing-rule-that-will-impact-every-ios-developer-1b0p)
- [App Review Guidelines — Apple Developer](https://developer.apple.com/app-store/review/guidelines/) (2.5.2; 4.7; 5.1.2)
- [Foundation Models framework — Apple Developer](https://developer.apple.com/documentation/FoundationModels)
