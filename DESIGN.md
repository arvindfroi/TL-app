# DESIGN.md — design language

> **Status: DRAFT / not yet defined.** This is a scaffold + a decisions checklist, not the final design language. The full language (colors, type scale, components, motion) gets created in a dedicated design session — a weekly reminder is set until it's done. Until then, build with **Apple system components and default styling**; don't invent bespoke visuals yet.

---

## 1. Intent

The TL app is a private, playful space for 9 friends (WAD?FC) — it should feel **personal and fun**, but stay **clean, fast, and native**. Two things are non-negotiable across every screen (see also `AGENTS.md` §8):

- **Security & privacy are visible, not hidden.** Closed group, Sign in with Apple, minimal data, clear "who can see this." Never a dark pattern.
- **No ads, ever. No ad SDKs, no tracking-for-ads, no sponsored content.** Nothing in the UI should ever be built to host an ad slot.

## 2. Platform & toolkit (current direction — subject to change)

- **SwiftUI + Apple's system UI** (native navigation, SF Symbols, system materials, sheets, lists).
- **Why:** fastest path, native feel, deep iPhone integration (Widgets, Live Activities, Siri), and you get **Dynamic Type, Dark Mode, VoiceOver, and accessibility largely for free**.
- **Subject to change:** if we want a stronger custom visual identity, we can layer custom components/themes on top of SwiftUI without leaving it. Decision point in the design session.

## 3. Typography — the dynamic / variable-font idea

This is the part you care most about: **fonts whose shape changes with the design** — stretching wider, thickening, shifting optical size, even animating.

**This is possible on iOS today** via **variable fonts**. A variable font exposes continuous *axes* you can set (and animate) at runtime:

- `wght` — weight (thin → black) — your "thicken"
- `wdth` — width — your "stretch" (condensed → expanded)
- `opsz` — optical size (adjusts shapes for display vs body)
- `slnt` / `ital` — slant / italic
- plus any **custom axes** a font defines (e.g. softness, "wonk," grade)

How it's done in SwiftUI: set variation axes on a `UIFontDescriptor` (via `kCTFontVariationAttribute`) and wrap it as a SwiftUI `Font`; animate by interpolating axis values across state changes. Helper packages exist: [swift-variablefonts](https://github.com/frzi/swift-variablefonts) and [VFont](https://github.com/dufflink/vfont). Apple's own **SF Pro** already ships width variants (Condensed/Expanded) and optical sizes. ([Font weight animation in SwiftUI](https://designcode.io/swiftui-handbook-font-weight-animation/))

**Decisions for the design session:**

- Pick the typeface(s). Want rich axes? Strong variable candidates to evaluate (decide later, don't assume): **Recursive**, **Roboto Flex** (many axes incl. width/grade/optical), **Fraunces** (optical + soft + "wonk"), **Inter**, or stay with **SF Pro**. Licensing must allow app embedding.
- Where dynamic behavior applies (headlines/branding that stretch & thicken) vs where it stays stable (body text for readability).
- **Always also support Dynamic Type** so text scales for accessibility — variable-axis styling and Dynamic Type must coexist.
- **Respect Reduce Motion**: animated font axes must tone down/stop when the user has Reduce Motion on.

## 4. To define in the design session (checklist)

- [ ] **Color** — brand palette (the WAD?FC green/blue/purple energy?), semantic tokens (background/surface/label/accent), full **light + dark** sets.
- [ ] **Typography** — chosen variable typeface(s), the type scale, which axes animate where, Dynamic Type mapping.
- [ ] **Spacing & layout** — base spacing scale (e.g. 4/8 pt), corner radii, grid.
- [ ] **Iconography** — SF Symbols as default; any custom marks (the WAD?FC logo usage).
- [ ] **Motion** — animation style (springs), the variable-font animations, Reduce-Motion behavior.
- [ ] **Components** — buttons, cards, chat bubbles, event cards, poll UI, vlog/story viewer, leaderboard, empty states.
- [ ] **App icon & launch** — finalize the WAD?FC icon.
- [ ] **Accessibility pass** — contrast (WCAG AA), Dynamic Type, VoiceOver labels, Reduce Motion, larger tap targets.
- [ ] **Tokens → code** — how tokens live in the codebase (a Swift `Theme`/asset catalog) so design and code stay in sync.

## 5. How we'll capture it

Likely a quick **Figma** (or Sketch) file for palette + type + key screens, then mirror the tokens into a Swift `Theme` (colors in the asset catalog, a typography helper for the variable axes). When that exists, link it here and update `AGENTS.md` §6 (Build & run) and the repo map.

---

_When the design language is finalized, replace the DRAFT banner, fill in the sections, and close the design reminder._
