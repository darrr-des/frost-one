# Frost One — Component Assessment Guidelines

A framework for evaluating the quality, completeness, and production-readiness of Frost One Design System components — across design, development (SwiftUI & Compose), and accessibility.

---

## Table of Contents

1. [What Defines a Good Component](#1-what-defines-a-good-component)
2. [Single Responsibility](#2-single-responsibility)
3. [Level of Abstraction](#3-level-of-abstraction)
4. [Canonical Name & Behavior Contract](#4-canonical-name--behavior-contract)
5. [Platform-Specific Idioms](#5-platform-specific-idioms)
6. [Minimum Shared API Surface](#6-minimum-shared-api-surface)
7. [Configuration Hierarchy](#7-configuration-hierarchy)
8. [Misuse-Resistant API](#8-misuse-resistant-api)
9. [Component Versioning](#9-component-versioning)
10. [Figma-to-Code Mapping (Code Connect)](#10-figma-to-code-mapping-code-connect)
11. [Component States & Source of Truth](#11-component-states--source-of-truth)
12. [Variants & Auto-Layout Structure](#12-variants--auto-layout-structure)
13. [Sensible Defaults](#13-sensible-defaults)
14. [Accessibility Requirements](#14-accessibility-requirements)
15. [Measuring Designer Effort Reduction](#15-measuring-designer-effort-reduction)

---

## 1. What Defines a Good Component

> A good component does one thing well, is named clearly, covers all necessary states, is accessible by default, uses design tokens, and is documented.

### Criteria

| # | Criterion | Key Question |
|---|---|---|
| 1 | **Single Responsibility** | Does it do one thing well? |
| 2 | **Consistent Naming** | Is it named for what it is, not how it looks? |
| 3 | **Variant Coverage** | Does it cover all states, sizes, and types? |
| 4 | **Accessibility** | Does it meet WCAG AA out of the box? |
| 5 | **Scalability & Flexibility** | Can it adapt to different contexts without breaking? |
| 6 | **Proper Use of Tokens** | Are colors, spacing, and typography tokenized? |
| 7 | **Documentation** | Is it self-explanatory with usage notes? |

---

## 2. Single Responsibility

> **Question:** What exact problem does this component solve — and what does it explicitly not do?

### How to Assess

- State the component's **one job** in a single sentence
- List what it **intentionally excludes** and why
- Flag any responsibility that belongs to a parent/molecular component

### Checklist

```
□ Can the component's purpose be described in one sentence?
□ Does it avoid owning label, validation, or group logic?
□ Are all states covered within that single responsibility?
□ Is there any missing state that falls within its responsibility?
```

### Common Gaps to Flag

| Gap | Signal |
|---|---|
| Missing indeterminate state | Incomplete binary selection coverage |
| Validation/error built in | Violates single responsibility — belongs to form level |
| Label text inside atomic component | Should live in molecular (e.g. CheckboxItem) |

---

## 3. Level of Abstraction

> **Question:** Is this component atomic, molecular, or organism — and is it at the right level?

### The Three Levels

| Level | Definition | Examples |
|---|---|---|
| **Atomic** | Single-purpose, no component children | Button, Checkbox, Icon, Badge |
| **Molecular** | 2–3 atoms forming a functional unit | Checkbox + Label, Input + Helper Text |
| **Organism** | Complete UI section with content logic | Form, Card, Navigation, Modal |

### Decision Questions

1. Can it be reused in completely different contexts?
2. Does it carry meaning without a parent?
3. Does combining atoms create something greater than the sum of its parts?

### Pitfalls

| Pitfall | Problem |
|---|---|
| Too atomic | Designers manually assemble the same pattern repeatedly |
| Too abstract | Component is too opinionated to reuse elsewhere |
| Wrong level | Molecular work in an atomic structure (or vice versa) |

---

## 4. Canonical Name & Behavior Contract

> **Question:** How do we ensure SwiftUI and Compose implementations are functionally equivalent without being identical?

### Naming Convention

| Layer | Format | Example |
|---|---|---|
| Figma | PascalCase | `Checkbox` |
| SwiftUI | PrefixedPascalCase | `FrostCheckbox` |
| Compose | PrefixedPascalCase | `FrostCheckbox` |
| Web | kebab-case | `frost-checkbox` |
| Tokens | dot.notation | `checkbox.border.enabled` |

### Behavior Contract

Define platform-agnostic rules — not implementation instructions:

- "When tapped, toggles between Selected and Unselected"
- "When disabled, ignores all interaction"
- "Focused state must be visually distinct and keyboard-accessible"

### Equivalence Table

| Behavior | Must be equivalent | Can differ |
|---|---|---|
| Toggle on interaction | ✅ | Animation mechanism |
| Disabled state | ✅ | How it's passed (`.disabled()` vs `enabled=false`) |
| Focused state | ✅ | Focus ring style |
| Accessibility label | ✅ | API syntax |
| Touch target size | ✅ (by platform minimum) | Exact value (44pt vs 48dp) |

---

## 5. Platform-Specific Idioms

> **Question:** How do we handle iOS haptics vs. Android ripple while preserving consistent design intent?

### The Separation Rule

| Layer | Owned by | Example |
|---|---|---|
| **Design Intent** | Designer | "Provide immediate feedback on interaction" |
| **Platform Idiom** | Developer | iOS: haptic / Android: ripple / Web: cursor change |

### Common Idiom Pairs

| Design Intent | iOS | Android | Web |
|---|---|---|---|
| Selection feedback | Haptic (`.selectionChanged`) | Ripple (`rememberRipple()`) | Visual fill animation |
| Hover | N/A | N/A | Border/color shift |
| Focus | Focus ring | Focus ring | Focus ring |
| Disabled tap | No haptic, no animation | No ripple, no animation | `cursor: not-allowed` |

### Non-Negotiable vs. Flexible

| Non-Negotiable | Flexible |
|---|---|
| Selected state must be visually distinct | How the transition animates |
| Disabled state suppresses all feedback | Mechanism of suppression |
| Focused state must be visible | Ring style |
| Minimum touch target size | Exact value per platform |

---

## 6. Minimum Shared API Surface

> **Question:** What props/parameters must this component expose on both platforms?

### The Three Categories

| Category | What it covers |
|---|---|
| **State Props** | What the component currently is |
| **Behavior Props** | What the component can do |
| **Accessibility Props** | What it communicates to assistive tech |

### Checkbox Minimum API

```
Checkbox(
  isSelected: Boolean,          // required
  onToggle: () -> Unit,         // required
  accessibilityLabel: String,   // required
  isDisabled: Boolean,          // optional — default: false
  isIndeterminate: Boolean,     // optional — default: false
  size: Size,                   // optional — default: medium
  accessibilityHint: String?    // optional — default: null
)
```

### What Must NOT Be in the API

| Excluded | Why |
|---|---|
| Color / fill | Comes from design tokens |
| Border style | Controlled by state + tokens |
| Animation duration | Platform idiom |
| Label text | Belongs to molecular component |
| Padding / spacing | Layout concern |

---

## 7. Configuration Hierarchy

> **Question:** Which props are required, optional, and which should never be exposed publicly?

### The Three Tiers

| Tier | Definition | Rule |
|---|---|---|
| **Required** | Component cannot function without it | No default possible |
| **Optional** | Has a sensible default | Document the default behavior |
| **Private / Internal** | Implementation detail | Never expose — can change without notice |

### Hierarchy Structure

```
Component
│
├── REQUIRED
│   ├── isSelected / value
│   ├── onToggle / onChange
│   └── accessibilityLabel
│
├── OPTIONAL
│   ├── isDisabled = false
│   ├── isIndeterminate = false
│   ├── size = .medium
│   └── accessibilityHint = null
│
└── PRIVATE
    ├── fillColor         ← derived from state + tokens
    ├── borderColor       ← derived from state + tokens
    ├── cornerRadius      ← fixed design token
    ├── animationSpec     ← platform idiom
    ├── focusRing         ← accessibility API
    └── touchTargetSize   ← platform minimum
```

### Golden Rules

1. Start with the least possible surface
2. Defaults should represent the 80% use case
3. Never expose styling props directly
4. Private props protect future refactors
5. Required props mean zero assumptions about context

---

## 8. Misuse-Resistant API

> **Question:** How do you make the correct usage obvious and incorrect usage impossible?

### Techniques

| Technique | Catches at | Use for |
|---|---|---|
| Enum / Sealed class | Compile time | Mutually exclusive states |
| Non-optional callback | Compile time | Required behavior |
| `require()` assertion | Runtime (early) | Invalid combinations |
| No styling props | Design time | Token consistency |

### State Enum Pattern

```kotlin
// ❌ Fragile — 16 possible combinations
isEnabled: Boolean, isDisabled: Boolean, isFocused: Boolean, isHovered: Boolean

// ✅ Safe — only one state at a time
sealed class CheckboxState {
  object Enabled    : CheckboxState()
  object Hovered    : CheckboxState()
  object Focused    : CheckboxState()
  object Disabled   : CheckboxState()
}
```

### Size Enum Pattern

```kotlin
// ❌ Raw value — ambiguous
FrostCheckbox(size = 12)

// ✅ Enum — intentional and clear
enum class CheckboxSize { Small, Medium }
FrostCheckbox(size = CheckboxSize.Medium)
```

### Misuse Risk Matrix

| Risk | Technique |
|---|---|
| Multiple states active | Enum / Sealed class |
| Invalid size value | Enum |
| Missing callback | Non-optional function type |
| Conflicting state flags | `require()` assertion |
| Color override | No styling props |

---

## 9. Component Versioning

> **Question:** How do code updates avoid breaking existing consumers on either platform?

### Semantic Versioning

| Bump | When | Example |
|---|---|---|
| **Patch** `1.0.x` | Bug fixes, visual tweaks | Fix disabled opacity |
| **Minor** `1.x.0` | New optional props, new variants | Add `isIndeterminate` |
| **Major** `x.0.0` | Removed/renamed props, restructured API | Rename `Type` to `isSelected` |

### Breaking Change Reference

| Change | Breaking? |
|---|---|
| Removing a prop | ✅ Yes |
| Renaming a prop | ✅ Yes |
| Adding a required prop | ✅ Yes |
| Renaming a Figma variant property | ✅ Yes |
| Removing a Figma variant | ✅ Yes |
| Adding an optional prop with default | ❌ No |
| Adding a new variant | ❌ No |
| Changing internal implementation | ❌ No |

### Deprecation Process

```
1. Mark old API as @Deprecated (Minor bump)
2. Run old + new API in parallel for one release cycle
3. Remove in next Major bump
4. Publish migration guide before removal
```

### Versioning in Figma

```
📄 Components          ← stable, current
📄 Components v2       ← next major, in review
📄 Archive             ← deprecated, do not use

Annotation frame per component:
Checkbox — Changelog
v1.2  Added isIndeterminate variant
v1.1  Added Medium size
v1.0  Initial release
```

---

## 10. Figma-to-Code Mapping (Code Connect)

> **Question:** How do we map Figma component properties 1:1 to SwiftUI/Compose parameters so Code Connect generates accurate snippets?

### Mapping Rules

| Rule | Requirement |
|---|---|
| Property names must match exactly | `Type` in Figma = `isSelected` in code — agreed upfront |
| Variant values must match enum cases | `Enabled` in Figma = `.enabled` in code |
| 2-value variants map to Boolean | `Selected/Unselected` → `isSelected: Bool` |
| 3+ value variants map to Enum | `Enabled/Hovered/Focused/Disabled` → `CheckboxState` |

### Complete Mapping Table

| Figma Property | Figma Values | Code Parameter | Code Type |
|---|---|---|---|
| `Type` | `Selected`, `Unselected` | `isSelected` | `Boolean` |
| `State` | `Enabled`, `Hovered`, `Focused`, `Disabled` | `state` | `CheckboxState` enum |
| `Size` | `Small`, `Medium` | `size` | `CheckboxSize` enum |

### What Breaks Code Connect

| Issue | Impact |
|---|---|
| Figma property name ≠ code parameter name | Snippet shows wrong prop |
| Figma value ≠ enum case | Snippet maps to wrong state |
| Missing Figma variant | Snippet can't represent that state |
| Spaces in variant values | Breaks enum generators |

### Design Responsibilities

- Use exact prop names that match code — agreed before building
- Never use spaces in variant values
- Document every property in the component spec
- Align naming before building — renaming after Code Connect = breaking change

---

## 11. Component States & Source of Truth

> **Question:** Which states must exist in both design and code, and who owns the source of truth?

### Universal State Checklist

```
□ Default (Enabled/Unselected)    — always required
□ Hovered                         — required (desktop/web)
□ Focused                         — required (keyboard/accessibility)
□ Pressed                         — required (mobile + desktop)
□ Selected / Active               — required (for toggleable components)
□ Disabled (Unselected)           — required
□ Disabled (Selected)             — required
□ Error                           — required (form context)
□ Loading                         — context-dependent
□ Indeterminate                   — required (select-all patterns)
```

### Ownership Model

| Aspect | Owner |
|---|---|
| Visual definition of states | Design (Figma) |
| When states are triggered | Code |
| State completeness | Shared |
| State naming | Shared |
| Accessibility of states | Shared |

### Source of Truth Hierarchy

```
Product Requirements
        │
        ▼
   Design (Figma) ← visual source of truth
        │
        ▼
   Design Tokens ← state values (colors, opacity)
        │
        ▼
   Code (SwiftUI / Compose) ← behavioral source of truth
        │
        ▼
   Code Connect ← links both, surfaces gaps
```

### What Happens When States Are Missing

| Scenario | Consequence |
|---|---|
| Missing in Figma, exists in code | Inconsistent pressed/error states across product |
| Missing in code, exists in Figma | Beautiful design that never ships |
| Missing in both | Component feels broken, accessibility fails |

---

## 12. Variants & Auto-Layout Structure

> **Question:** How do you structure variants and auto-layout so resizing and state switching require zero manual adjustment?

### Variant Structure Rules

| Rule | Requirement |
|---|---|
| Identical layers across all variants | Only visibility and color change between variants |
| Never add/remove layers between variants | Toggle visibility instead |
| Consistent layer naming | Same name in every variant |
| Property order: most-used first | Type → State → Size |
| No spaces in variant values | Breaks Code Connect and search |

### Auto-Layout Rules

| Rule | Implementation |
|---|---|
| Every frame uses auto-layout | No fixed frames for content containers |
| Atomic components: Hug × Hug | Always exactly the size of content |
| Molecular+: Fill width, Hug height | Adapts to container, grows with content |
| Padding in container, not child | Keeps atoms pure |
| Text never fixed width (unless intentional) | Prevents clipping at large text sizes |

### The Zero-Adjustment Test

```
State Switching Test:
□ Switch Type: Selected → Unselected  → No layout shift?
□ Switch State: Enabled → Disabled    → No movement?
□ Switch Size: Small → Medium         → Everything scales?

Resizing Test:
□ Stretch horizontally                → Content reflows?
□ Shrink below minimum                → Text truncates cleanly?
□ Place in tight container            → No overflow?
```

---

## 13. Sensible Defaults

> **Question:** How do you define defaults so the component is production-ready with zero overrides?

### A Sensible Default Must

| Criteria | Question |
|---|---|
| Represent the 80% use case | Is this how it's used most of the time? |
| Be visually complete | Does it look finished, not like a wireframe? |
| Be functionally safe | Does it avoid accidental destructive behavior? |
| Be accessible out of the box | Does it meet WCAG without configuration? |

### Default Stack

| Category | Property | Default | Why |
|---|---|---|---|
| State | `isSelected` | `false` | Checkboxes start unchecked |
| State | `isDisabled` | `false` | Assume interactive |
| State | `isIndeterminate` | `false` | Special case only |
| Size | `size` | `medium` | Most versatile, meets touch targets better |
| Color | Fill | Token: `checkbox.fill.selected` | Never hardcode |
| Color | Border | Token: `checkbox.border.default` | Adapts to themes |
| Shape | Corner radius | Token: `radius.checkbox` | Consistent with design language |
| Layout | Touch target padding | 16px (extends to 44/48dp) | Accessibility by default |
| Content | `accessibilityLabel` | Required — no default | Cannot be assumed |
| Behavior | `onToggle` | Required — no default | No handler = broken |

### The Production-Ready Default Test

```
Drop the component with zero configuration:
□ Does it look like a real, finished UI element?
□ Does it show the most common use case?
□ Is every color from a token (not hardcoded)?
□ Is it accessible without extra padding?
□ Is it interactive (not accidentally disabled)?
□ Would a developer recognize it as correct?
□ Would a screen reader describe it meaningfully?
```

---

## 14. Accessibility Requirements

> **Question:** What accessibility requirements are non-negotiable in the design?

### The Three Non-Negotiable Pillars

| Pillar | iOS | Android | Web |
|---|---|---|---|
| Screen Reader | VoiceOver | TalkBack | NVDA / JAWS |
| Dynamic Sizing | Dynamic Type | Font Scale | Browser text zoom |
| Touch Target | 44×44pt minimum | 48×48dp minimum | 24×24px (WCAG 2.5.5) |

---

### Pillar 1 — Screen Reader

Every interactive component must communicate:

| Signal | What it answers | Example |
|---|---|---|
| **Role** | What kind of element is this? | "Checkbox" |
| **Label** | What is it for? | "Accept terms and conditions" |
| **Value** | What is its current state? | "Checked" / "Unchecked" |
| **Hint** | How do I interact with it? | *(platform auto-generates)* |

**Non-Negotiable Rules:**
- Every state must trigger an announcement
- Labels must never be empty or generic ("checkbox" is not a label)
- State changes must announce immediately on toggle
- Disabled components must still be reachable by screen readers

---

### Pillar 2 — Dynamic Type / Font Scale

**Non-Negotiable Rules:**
- No fixed height on containers that hold text
- Text must never truncate without explicit intent
- The component box stays fixed — only surrounding text scales
- Test at default, AX3 (~24pt / 130%), and maximum (AX5 / 200%)

**Minimum Text Sizes:**

| Element | Minimum |
|---|---|
| Label | 14–16pt/sp |
| Helper text | 12pt/sp |
| Error text | 12pt/sp |

---

### Pillar 3 — Minimum Touch Target

**Hard Numbers:**

| Platform | Minimum | Source |
|---|---|---|
| iOS | 44×44pt | Apple HIG |
| Android | 48×48dp | Material Design |
| Web | 24×24px | WCAG 2.2 AA |
| Web (best practice) | 44×44px | Matches iOS HIG |

**Solution — Invisible Touch Target Layer in Figma:**
```
□ Add transparent layer: 44×44pt minimum
□ Name it "Touch Target" or "Hit Area"
□ Set opacity to 0
□ Lock it (prevent accidental moves)
□ Document: "Defines tap area, not visual size"
```

---

### Full Accessibility Checklist

```
Screen Reader:
□ Role defined?
□ accessibilityLabel required in API?
□ State announced on change?
□ Disabled state still accessible?
□ Indeterminate state has announcement?

Dynamic Type:
□ Checkbox box fixed size (correct)?
□ Container uses auto-layout?
□ No fixed-height text containers?
□ Tested at 200% scale?

Touch Target:
□ Visual size meets minimum? (or padded to meet it)
□ Touch target layer in Figma?
□ Touch target documented in spec?

Color & Contrast:
□ Text on background ≥ 4.5:1 (WCAG AA)?
□ UI components ≥ 3:1 (WCAG AA)?
□ Focus indicator ≥ 3:1 against adjacent colors?
```

### Ownership

| Requirement | Design | Code |
|---|---|---|
| Visual contrast ratio | ✅ | |
| Touch target size definition | ✅ | |
| State visual differentiation | ✅ | |
| Dynamic type layout | ✅ | |
| accessibilityLabel value | ✅ defined | ✅ implemented |
| Role assignment | | ✅ |
| State announcement | | ✅ |
| Focus management | | ✅ |

---

## 15. Measuring Designer Effort Reduction

> **Question:** How do you measure whether this component is reducing designer effort over time?

### The Four Metrics

| Metric | Formula | Target |
|---|---|---|
| **Adoption Rate** | Component instances ÷ Total instances of that pattern | > 85% |
| **Override Rate** | Overridden instances ÷ Total instances | < 10% |
| **Detach Rate** | Detached instances ÷ Total instances | < 5% |
| **Time Savings** | Time from scratch − Time with component | Track per sprint |

---

### Override Classification

| Type | Signal | Action |
|---|---|---|
| Expected (label text) | ✅ Normal | None |
| Tolerated (1–2px nudge) | ⚠️ Minor friction | Investigate |
| Concerning (color changed) | 🚩 Variant gap | Add variant |
| Breaking (structure modified) | 🚩 Use case not covered | Extend API |
| Detach | 🚩 Critical failure | Audit and rebuild |

---

### Override Root Cause Analysis

```
Why did the designer override this?
          │
          ├── "The variant I needed didn't exist"     → Add variant
          ├── "The default wasn't right"               → Reconsider default
          ├── "I didn't know this variant existed"     → Documentation gap
          ├── "The component couldn't do what I needed"→ API gap
          └── "I was in a hurry"                       → Culture problem
```

---

### Quarterly Health Dashboard

| Metric | Target | Cadence |
|---|---|---|
| Adoption rate | > 85% | Monthly |
| Override rate (concerning+) | < 10% | Monthly |
| Detach rate | < 5% | Monthly |
| Missing variant reports | 0 | Per sprint |
| Designer satisfaction (1–5) | > 4 | Quarterly |
| Time to configure | < 60 seconds | Quarterly |
| Code-design parity issues | 0 | Per release |

---

### The Feedback Loop

```
Component ships
      │
      ▼
Measure adoption + overrides (monthly)
      │
      ├── Overrides cluster around one property? → Add variant
      ├── Detach rate rising?                    → Roadmap fix
      ├── Adoption stalling?                     → Docs problem
      └── Time savings confirmed?                → Justify investment
```

---

## Using This Framework

### Assessment Scorecard

For each component, score each section:

| Score | Meaning |
|---|---|
| ✅ Pass | Meets the requirement fully |
| ⚠️ Partial | Partially meets — needs improvement |
| 🚩 Fail | Does not meet — action required |
| ❓ Unknown | Needs investigation |

### Assessment Order

Run assessments in this order for maximum efficiency:

1. **Single Responsibility** — establishes scope for everything else
2. **Level of Abstraction** — confirms the component is at the right level
3. **Component States** — uncovers the full surface area to assess
4. **Accessibility** — non-negotiable baseline
5. **Variants & Auto-Layout** — structural integrity
6. **Sensible Defaults** — production readiness
7. **API Surface & Configuration Hierarchy** — developer contract
8. **Figma-to-Code Mapping** — Code Connect readiness
9. **Versioning** — release strategy
10. **Platform Idioms & Behavior Contract** — cross-platform parity
11. **Misuse-Resistant API** — API safety
12. **Canonical Name** — naming alignment
13. **Measuring Effort Reduction** — ongoing health

---

*Frost One Design System — Component Assessment Framework*
*Maintained by the Frost Design Group*
