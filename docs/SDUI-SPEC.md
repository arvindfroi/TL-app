# SDUI-SPEC — the server-driven layout engine

> **Status: design spec (proposed, June 2026).** Defines how groups customize *layouts*, not just colors. Companion to [`PIVOT.md`](PIVOT.md) and [`RULES-SPEC.md`](RULES-SPEC.md). Implementation lands in roadmap Phase 2.
>
> **Performance is non-negotiable here:** this engine is the main jank/bloat risk, so it must follow [`PERFORMANCE.md`](PERFORMANCE.md) — documents **compile once into typed Swift** (no JSON on the render hot path), render via granular bindings + lazy collections, and default templates ship precompiled. A customized screen must cost the same as a hardcoded one.

## 1. Goal and the non-negotiable constraint

Let each group rearrange, add, remove, and restyle the screens of their app — far beyond colors and fonts — **without shipping a new build, and without violating App Store Guideline 2.5.2.**

The constraint shapes everything: **the renderer is an interpreter compiled into the app binary; only declarative *data* is downloaded.** A layout is a document, not code. We deliberately render **native SwiftUI from data** and avoid the HTML5/JavaScript "mini-app" route (Guideline 4.7), which would impose an index/disclosure regime and forbid native-API bridges.

## 2. The three customization layers

Customization is a stack; a group can stop at any level.

1. **Theme tokens** *(easiest, Phase 1)* — semantic design variables: color roles (background/surface/label/accent/…), the type scale + which **variable-font axes** animate where (the original DESIGN.md dynamic-type idea survives here), spacing scale, corner radii, motion. Changing a token restyles the whole app instantly.
2. **Component variants & section toggles** *(friendly)* — pick a card style, show/hide sections, reorder a screen's sections, choose list vs. grid. Bounded choices, hard to break.
3. **Full layout documents (SDUI)** *(advanced)* — author the screen tree directly: any component, any nesting, any data binding. Maximum power; gated and validated.

## 3. The layout document

A screen is a versioned JSON tree. Conceptual shape:

```jsonc
{
  "schema": 1,                       // engine schema version (forward-compat)
  "screen": "home",                  // logical screen id the app asks for
  "theme": "wadfc-default",          // theme token set to resolve against
  "root": {
    "type": "VStack",                // a component from the embedded registry
    "props": { "spacing": "md" },    // props resolve tokens (e.g. "md" -> 12pt)
    "children": [
      { "type": "Header", "props": { "title": "{group.name}" } },
      { "type": "EventCard",
        "bind": { "source": "events.next" },          // data binding
        "props": { "style": "compact", "showRSVP": true } },
      { "type": "List",
        "bind": { "source": "chat.recent", "limit": 20 },
        "item": { "type": "MessageRow" } },           // per-item template
      { "type": "Conditional",
        "when": "role in [owner, admin]",             // RBAC-aware visibility
        "child": { "type": "Button",
                   "props": { "title": "New event" },
                   "action": { "intent": "createEvent" } } }
    ]
  }
}
```

Key ideas: **`type`** names a registry component; **`props`** are styling/config (token-resolvable); **`bind`** connects a component to a domain data source; **`item`** is the template for each row of a bound list; **`action`/`intent`** dispatches to a fixed set of app intents (never arbitrary code); **`when`** gives RBAC- and state-aware conditional rendering.

## 4. The component registry (the safety boundary)

The renderer can only instantiate components from a **fixed registry baked into the binary**. A layout naming an unknown component does **not** execute anything new — it renders the registered component or a graceful placeholder. The registry is the allow-list that keeps "data, not code" true.

Seed registry (extend over time): **Layout** — `VStack`, `HStack`, `ZStack`, `Grid`, `List`, `ScrollView`, `Spacer`, `Conditional`, `TabContainer`. **Content** — `Text`, `Image`, `Header`, `Divider`, `Badge`. **Domain** — `MessageRow`, `ChatComposer`, `EventCard`, `RSVPControl`, `PollCard`, `AvailabilityCalendar`, `Leaderboard`, `MemberList`, `WishlistItem`. **Controls** — `Button`, `Toggle`, `SegmentedControl`, `MenuButton`.

Each registry component declares its allowed props, the data shape it binds to, and its default styling — this metadata also powers the editor and validation.

## 5. Data binding

Bindings reference a small, documented **data-source catalog** (`events.next`, `chat.recent`, `members.all`, `polls.open`, `availability.overlap`, `points.leaderboard`, `group.*`, `me.*`). Each source returns a typed shape the component knows how to render. Templating tokens like `{group.name}` interpolate read-only values into text props. Bindings are **read-through the permission layer** — a binding can never return data the current member's role isn't allowed to see, regardless of what the layout asks for.

## 6. Theming tokens

Tokens are a named map resolved at render time, so the same layout looks different per theme. Categories: **color roles** (semantic, with full light + dark sets), **typography** (scale steps + variable-font axis values, mapped to Dynamic Type for accessibility), **spacing** (`xs…xxl`), **radii**, **motion** (durations/curves, honoring Reduce Motion). The existing [`DESIGN.md`](../DESIGN.md) becomes the *default theme*, not the only one. Accessibility floors (contrast, Dynamic Type, tap targets, Reduce Motion) are **enforced by the renderer** so a custom theme can't make the app unusable.

## 7. Versioning, validation, and graceful degradation

- **Schema version** on every document; the renderer supports a range and migrates older docs forward.
- **Unknown component / prop / binding → graceful fallback** (render nothing, a labeled placeholder, or the default for that screen), never a crash. This is what lets an older app open a newer layout safely.
- **Validation on save** (in the editor) and **defensive validation on render**: type checks, registry membership, binding existence, RBAC of any `when`/`action`, and accessibility floors.
- **Every screen has a built-in default layout** shipped in the binary; if a group's custom layout is missing, corrupt, or fails validation, the app falls back to the default and surfaces a fixable error to admins.

## 8. Storage and sync

Layout documents, theme token sets, and the choice of which layout is active per screen live in the **group's CloudKit zone** (the customization store from [`ARCHITECTURE.md`](ARCHITECTURE.md)), so they sync to every member and respect the zone's access boundary. Editing follows the same near-real-time (not live co-edit) model as the rest of the app; last-write-wins with an edit lock hint for layouts being actively edited.

## 9. The editor (friendly + advanced)

- **Friendly mode:** a visual editor over the *section/variant* layer — reorder sections, toggle them, pick component variants, edit theme tokens with live preview. Bounded so non-technical members can't break the app.
- **Advanced mode:** direct layout-document editing with live validation and preview, gated to members whose role permits it (RBAC). This is the "advanced settings" surface for layouts, paired with the rules-engine advanced surface.
- **Preview & rollback:** preview before publish; keep the previous version so a bad layout is one tap to revert.
- **Templates:** export a screen or whole-app setup as a shareable template (feeds the Phase 4 template gallery); import validates against the registry before applying.

## 10. Permissions (who can change what)

Driven by the RBAC layer: **owner/admin** can edit layouts and themes and publish to the group; **members** may be allowed personal overrides if the group enables it; **invited** users get the default. Every editor action is permission-checked in the data layer, not just hidden in the UI.

## 11. Cross-platform note (future)

Because the document is platform-neutral data and only the *renderer* is Apple-native, a future Android build (via Skip) can ship its own renderer over the **same documents** — no document format change. Keeping the registry semantically platform-neutral now is the cheap insurance that preserves that option.
