# AI-READINESS — designing the app so AI can help configure and script it

> **Status: design principle + forward plan (proposed, June 2026).** AI features are **down the line**, but the *content model* must be AI-ready **now** so it isn't a costly retrofit. Companion to [`SDUI-SPEC.md`](SDUI-SPEC.md), [`RULES-SPEC.md`](RULES-SPEC.md), [`ARCHITECTURE.md`](ARCHITECTURE.md), [`PERFORMANCE.md`](PERFORMANCE.md). Scope-limited by what App Store review allows (§6).
>
> ✅ **v1 scope (decided — [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L1):** customization in v1 is **settings/editor-first** (a creator surface where you choose options and *paste scripts/config*). **AI is a later connector, not a v1 feature.** This doc's job *now* is purely **futureproofing** — keeping customization as validated declarative documents so adding an AI client later is cheap. Don't build the assistant yet.

## 1. The thesis: AI-friendly by construction

We already made the decision that unlocks AI almost for free: **layouts, themes, configs, and rules/scripts are declarative documents with documented schemas, a fixed component registry, and a closed action allow-list** (data, not code — see [`DECISIONS.md`](DECISIONS.md) ADR-005). That same structure is what an AI needs to read and write the app.

So the policy is simple: **the customization schema *is* the AI's API.** An AI assistant is just **another client of the exact pipeline the in-app editor uses** — it proposes a document (or a diff), the engine validates it, the user previews and approves, and only then is it applied. The AI **never executes anything**; it emits data that the embedded, sandboxed engines validate like any other. This is simultaneously the safety model and the App Store-compliance model.

> One line: **the editor and the AI are two clients of one validated, data-only API. AI proposes; the engine validates; the user approves; nothing runs as code.** (In v1, the *editor* is the client we build; the AI client comes later.)

## 2. The interaction loop (the safety keystone)

Every AI action (when we build it) follows the same path the editor already uses:

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
- **Human-in-the-loop by default.** Changes are proposals until a permitted user approves.
- **Reversible.** Versioning + audit log mean any AI change is inspectable and one tap from undo.

## 3. Make the content AI-legible (do this now — it's the futureproofing)

These are cheap to build now, power the **editor** in v1, and are exactly what an AI client needs later:

- **Published schemas** (JSON Schema or equivalent) for each document type: layout, theme tokens, rule, group config. Versioned alongside the engine schema versions.
- **Capability manifest** — the machine-readable catalogs the engines already need: the **component registry** (allowed props + bound data shapes), the **data-source catalog**, and the **trigger / condition / action catalog**.
- **Stable IDs + semantic names + doc-strings.** Every component, token, source, trigger, and action carries a stable id, a human/semantic name, and a one-line description.
- **Self-describing errors.** Validation failures return structured, human-readable reasons so a future AI can repair its own proposal (and so the editor gives good messages now).

## 4. The AI connector abstraction (pluggable backends — later)

A single internal interface — the **assistant provider** — so backends are swappable:

- **On-device (default): Apple Foundation Models.** Free, private, no data leaves the device → **no extra consent burden** (§6). Gated to capable devices, graceful fallback. The preferred path for config/script help.
- **External connectors (opt-in): cloud LLMs.** Off by default, enabled per group/user with explicit consent + disclosure (§6).
- **Structured output / function-calling** maps the model onto the **capability manifest**: the model emits a document or a sequence of catalog operations, never free-form code. Output is re-validated regardless of provider.
- **Grounding + guardrails:** the provider gets the schemas + current document + the relevant data slice (minimized, permission-filtered) — never another group's data.

Ship the editor first; add this interface when AI gets built.

## 5. Apple Intelligence & Siri (via App Intents)

As of WWDC 2026, **App Intents is the only way Siri and Apple Intelligence reach into a third-party app** (SiriKit is deprecated). Same "expose capabilities as a structured catalog" pattern as our engines:

- **Expose core actions and content as App Intents** conforming to **assistant schemas** — invoked by natural language, content in the **Spotlight semantic index** with attribution. Wrapping actions as App Intents is worth doing as they're built (gives Siri/Shortcuts/widgets for free), even before the in-app AI.
- **Examples:** "Mark me home in Sandnes," "RSVP yes to Friday," "What's the next hangout?"
- **View Annotations** later enable on-screen references.
- **Always gated + graceful:** Apple Intelligence features limited to capable devices, **never a hard dependency**.

## 6. App Store guardrails (the hard limits)

- **2.5.2 — data, not code.** AI assistance produces **declarative documents**, interpreted by the embedded engines. The model never downloads or executes code; "write me a script" means "emit a validated rules document." Keeps us clear of 2.5.2 and the 4.7 mini-app regime.
- **5.1.2(i) — third-party AI consent (updated Nov 2025).** Before any personal/group data is sent to a **third-party AI**, **disclose which third party and why, and get explicit in-app consent**. External connectors are off by default, opt-in, scoped; **on-device Foundation Models need none of this**.
- **Privacy & tenancy.** Prefer on-device; minimize + permission-filter; **never send another group's data**; no training on user content without consent.
- **Device gating.** Apple Intelligence / Foundation Models require capable hardware + recent iOS; gate gracefully.
- **Non-negotiables still apply.** No ads, privacy-by-default — AI must not become a data-harvesting or ad vector.

## 7. What to build now vs. later

**Now (futureproofing — folds into the editor + engine work):**
- Documented, versioned **schemas** for every customization document.
- A **capability manifest** with stable IDs, semantic names, doc-strings, structured validation errors.
- One **programmatic create/edit/validate API** the in-app editor uses (an AI client reuses it verbatim later).
- Wrap core actions as **App Intents** as they ship.

**Later (its own phase, [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L1):**
- The **assistant-provider** interface + on-device Foundation Models (config/script help, "why didn't my rule fire," theme suggestions).
- **External connector(s)** with the 5.1.2(i) consent flow.
- **Siri / Apple Intelligence** assistant-schema adoption + View Annotations.

## 8. Concrete use cases this enables (later)

- *"Make our chat screen more compact and move events to the top."* → AI emits a **layout diff**; preview; approve.
- *"Set up a rule: remind everyone the night before any event."* → AI emits a **rules document**; validated; approve.
- *"Why didn't my weekend-poll rule fire last Saturday?"* → AI reads the **execution/audit log** and explains.
- *"Give us a cozy ski-trip theme."* → AI proposes **theme tokens** within accessibility floors.
- *"Hey Siri, mark me home in Sandnes."* → an **App Intent**, no in-app AI needed (can ship early).

## 9. Where this lands in the plan

Cross-cutting, mostly **later**, but with **two cheap obligations pulled in now**: (a) keep every customization surface schema-first with a programmatic, validated edit API (which the v1 editor needs anyway), and (b) wrap core actions as App Intents as they ship. Those make the later AI work additive instead of a rebuild. See [`OPEN-DECISIONS.md`](OPEN-DECISIONS.md) L1, [`DECISIONS.md`](DECISIONS.md) ADR-013, [`BACKLOG.md`](BACKLOG.md).

---

### Sources (verified June 2026)
- [Integrating actions with Siri and Apple Intelligence — Apple Developer (App Intents)](https://developer.apple.com/documentation/AppIntents/Integrating-actions-with-siri-and-apple-intelligence)
- [Making actions and content discoverable by Apple Intelligence — Apple Developer](https://developer.apple.com/documentation/appintents/integrating-your-app-with-siri-and-apple-intelligence)
- [Apple's new App Review Guidelines clamp down on apps sharing personal data with third-party AI — TechCrunch (Nov 2025)](https://techcrunch.com/2025/11/13/apples-new-app-review-guidelines-clamp-down-on-apps-sharing-personal-data-with-third-party-ai/)
- [Guideline 5.1.2(i) — the AI data-sharing rule (DEV)](https://dev.to/arshtechpro/apples-guideline-512i-the-ai-data-sharing-rule-that-will-impact-every-ios-developer-1b0p)
- [App Review Guidelines — Apple Developer](https://developer.apple.com/app-store/review/guidelines/) (2.5.2; 4.7; 5.1.2)
- [Foundation Models framework — Apple Developer](https://developer.apple.com/documentation/FoundationModels)
