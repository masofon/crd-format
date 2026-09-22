---
id: divider
name: Divider
version: 0.4.0
format: 2
lifecycle: active
status:
  requirements: complete
  design_specs: complete
  implementation: not_started
  brand_validation: complete
  cms_behaviour: complete
  docs_guidance: complete
  review: not_started
figma: [link to the component set — redacted for sharing]
storybook:
related_crds:
  dependency: []
  precedent: [list]
last_updated: 2026-09-22
---

# CRD: Divider

---

# 1 · Requirements

## Purpose

Separates groups of related items inside a single container with a hairline rule, so grouping is
visible without adding a heading, a nested container or extra spacing.

It is the smallest structural atom in the library: one visual part, no interaction, no content.

## Scope

**In scope:**

* A horizontal hairline rule spanning the width of its container
* Two padding behaviours, selected by the `inset` property
* Adapting colour to the scheme of the context it is placed in

**Out of scope:**

None.

## Anatomy

* `container` — the frame that holds the rule and owns Divider's own horizontal padding
* `rule` — the hairline itself; carries the thickness and the colour

## Functional requirements

* F1: `container` fills the width of its parent.
* F2: `rule` fills the width of `container`.
* F3: `rule` is the only child of `container`. Divider never contains text, icons or any other
  content.
* F4: `rule` takes its thickness and colour from the shared structural border tokens. Divider
  defines no component-tier token of its own.
* F5: `inset=true` applies horizontal padding to `container`; `inset=false` applies none.
* F6: `inset` defaults to `false`.
* F7: `container` applies no vertical padding at either `inset` value. Vertical space around a
  Divider belongs to the parent.
* F8: Divider is horizontal. There is no vertical orientation.

## States

* S1: Divider has no states. It has no hover, focus, pressed, selected, disabled or loading
  treatment.

## Behaviour

* B1: Divider has no interactive behaviour.
* B2: Divider has no motion.
* B3: Divider has no responsive behaviour of its own. `rule` thickness does not change with
  breakpoint.

## Content

None. Divider carries no text or media — see F3.

## Accessibility

* A1: Divider sets no ARIA role and is excluded from the accessibility tree. The grouping it
  depicts is conveyed by the structure of the parent, never by Divider.

## Integration

**Composition rules:**

* I1: Divider is placed as a sibling between the elements it separates. It never wraps or
  contains them.

**Children:**

None.

**Used in:**

* Context Menu (C1, interim — no CRD yet)

## Ruled out

* **A vertical orientation** — no evidenced need. A vertical rule needs its own inset story
  (padding on the block axis rather than the inline axis), and adding it later is additive rather
  than breaking.
* **A labelled or "or" divider** — would add a text style, alignment rules and a second token to
  a one-part atom. If a consumer needs one it becomes its own component, not an axis here.
* **An emphasis axis (subtle / strong)** — a second colour token with no consumer asking for it.
  Structural separation has one weight.
* **A thickness axis** — thickness is a single shared structural value. A per-instance thickness
  would let consumers diverge on a structural token.
* **Scheme-independent colour using the `locked-` variants** — Divider carries no status meaning,
  so it must adapt to the surface it sits on rather than hold its own colour.
* **`role="separator"`** — making the rule semantic would place the grouping in presentation. The
  parent's structure carries it instead. Consistent with List, whose CRD states it "intentionally
  avoids menu, listbox, or selection semantics".
* **A `semantic` property letting each consumer choose the role** — every consumer would then own
  an accessibility decision, and most would take the default unexamined.
* **Requiring FILL to be set by hand on each insert** — in code an instruction in a component
  description cannot be enforced, so filling is structural; see F1 and F2. In Figma the manual
  step is unavoidable outside a slot that stretches its children, so it survives there as a
  Figma-only caveat in the component description.
* **`inset=true` as the default** — the only current consumer owns its own horizontal padding, so
  `true` would be wrong at every use site and each insertion would need a manual override.

## Open questions

| Question | Owner | By | Status |
|---|---|---|---|

None open.

## Notes

**F1 and F2 are code-only.** Figma has no property that makes a component's instances fill their
container, so the component cannot declare it. Inside Context Menu it happens regardless, because
that slot sets `stretchChildOnInsert`; elsewhere in Figma it is a manual step on insert. The
Figma approximation is therefore "nothing" — the component set carries the caveat in its
description instead.

A heavier structural border colour exists, `border/{scheme}/structural/bold`. No Divider variant
exposes it — see Ruled out on the emphasis axis.

Divider has no `component/divider/*` token path; it consumes the shared structural border
tokens directly.

---

# 2 · Design specifications

## Properties in Figma

| Property | Type | Default | Options | Description |
|---|---|---|---|---|
| `scheme` | Variant | `base` | `base`, `alt`, `inverse`, `inverse-alt`, `expressive`, `expressive-alt` | Pick the scheme of the surface the Divider sits on. There is no scheme prop in code — a `[data-scheme]` ancestor remaps the token. |
| `inset` | Variant | `false` | `true`, `false` | `false` is full-bleed and inherits the parent's padding. `true` applies Divider's own horizontal padding, for parents that are themselves full-bleed. |

## Design decisions

* 2026-09-09: **Schemable, not scheme-independent.** A structural hairline carries no status
  meaning, so it must adapt to the surface it sits on. All six scheme variants already exist in
  C1 and the base variant binds base-branch tokens, so this records what is built rather than
  changing it.
* 2026-09-09: **Decorative, not `role="separator"`.** The grouping a Divider depicts should
  already exist in the parent's markup; making the rule semantic would move that grouping into
  presentation. Cites List's position that grouping carries no selection or menu semantics.
* 2026-09-09: **`inset` defaults to `false`, reversing the built default.** Context Menu, the only
  current consumer, owns its own horizontal padding — so `true` is wrong at every current use
  site, and a default that must be overridden on every insertion is the same class of assembly
  defect the Context Menu work exists to remove. Nothing breaks, since it is the only consumer, so
  no Migration section is needed.
* 2026-09-09: **Divider fills its container structurally, not by instruction.** The built
  component is fixed-width and its description asks for FILL to be set on insert. A hairline that
  does not span its container is always wrong, so this is made structural.
* 2026-09-09: **Promoted from interim to Core Components** after an audit against this CRD, not a
  rebuild. The interim build was already correct on anatomy, tokens and all six schemes, so
  promotion cost one variant-default change. Recorded because "interim" implies a rebuild is owed,
  and here it was not.
* 2026-09-09: **F1 and F2 are code-only.** Figma cannot express "instances always fill their
  container" — there is no such property — so the requirement stands for code and the Figma side
  relies on the consuming slot's `stretchChildOnInsert`, or a manual step. Stated because a build
  agent comparing Figma to the CRD would otherwise read the fixed-width variants as a defect.
* 2026-09-09: **The context card was reduced to a CRD pointer, changelog and contacts.** Its
  Description, Variants, Usage and Follow-ups sections restated this document, and had already
  drifted — the Variants section described `inset=true` as the Context Menu variant, the opposite
  of the truth. Someone could plausibly propose the opposite (that a card should be
  self-sufficient), and `resin-design-component` § 1 currently instructs exactly that, so the
  decision is logged rather than assumed.
* 2026-09-22: **Not available in the CMS.** Divider is a structural part of the components that
  use it rather than something a self-serve user places, so section 5 states its absence instead of
  listing exposed properties. Logged because a one-part atom is exactly the kind of thing someone
  would assume is CMS-exposable.
* 2026-09-22: **Usage examples are illustrative, not a closed list.** Where a boundary is needed is
  the designer's judgement, so "when to use" names places Divider works rather than the only places
  it is allowed. Logged because a bare bullet list reads as exhaustive unless it says otherwise.
* 2026-09-09: **The `inset` axis is kept rather than dropped.** Dropping it and making insetting
  always the parent's job was considered; the axis stays so a full-bleed container can inset
  without adding padding of its own.

---

# 3 · Implementation

## Implementation decisions

## Dev notes & deviations

---

# 4 · Brand validation

## Contrast pairings

**Token pattern:** Divider defines no component-tier token. The pairing is
`border/{scheme}/structural/default` against the background of whatever contains it —
`background/structure/{scheme}` for its only current consumer.

**Validated at brand creation:**

| Foreground token | Background token | Minimum ratio |
|---|---|---|
| `border/{scheme}/structural/default` | `background/structure/{scheme}` | None required — see Exemptions |

**Validated at placement (transparent fills):**

| Foreground token | Validated against | Minimum ratio |
|---|---|---|

Neither token is transparent, so there is nothing to validate at placement.

**Exemptions:**

`border/{scheme}/structural/default` against `background/structure/{scheme}` is exempt from WCAG
1.4.11 non-text contrast, because Divider is decorative and sets no ARIA role (A1).

The exemption is not a concession to current values. The structural border tokens are not tuned to
a contrast threshold and are expected to sit below 3:1 in some schemes and themes, which is correct
for a hairline separator.

It is also why A1 carries weight: a Divider that became semantic would need this pairing validated,
and the tokens would not pass as they stand. Anything that must be perceivable as a boundary needs
its own affordance rather than this rule.

Measured per-brand ratios: [tracker reference — redacted for sharing].

## Validation logic notes

Brand Builder should not add this pairing to its ratio checks. A passing threshold would force the
rule darker than a hairline separator should be, and the exemption above is the intended outcome
rather than a gap to close.

---

# 5 · CMS behaviour

Divider is not available in the CMS. It is a structural part of the components that use it, not a
block a self-serve user places, so there are no layout constraints, no exposed properties and no
guardrails to state.

---

# 6 · Documentation guidance

## Usage

**When to use:**

Divider is one way to show a boundary, not the only one, and whether a layout needs one is the
designer's call. Places it works:

* Between groups of related rows inside a single container, such as groups of rows in a Context
  Menu
* Inside a card, separating a header or footer from the body
* Between groups of fields inside one form

**When not to use:**

* To separate whole page or section regions from each other. Divider works inside a single
  container; boundaries between containers come from the layout's own surfaces and spacing.
* As a vertical rule between side-by-side elements. Divider is horizontal only (F8) — there is no
  vertical form, and rotating an instance is not one. If you need it, it is an amendment to this
  component.
* As a spacer. Divider is a boundary, not a gap — vertical space comes from the parent's spacing
  tokens (F7).
* As a visual accent or emphasis. Divider always spans its container (F1, F2) and has one weight,
  so it cannot be the short decorative rule this usually means; emphasis comes from surface,
  typography or spacing.
* As an underline. Divider always spans its container (F1, F2), so it cannot sit under a word or a
  heading; underlining belongs to the text style.

**Patterns:**

* Inside a container that already applies horizontal padding, use `inset=false` so the padding is
  applied once. Inside a full-bleed container, use `inset=true`.
* The space either side of a Divider comes from the parent's gap tokens (F7), and should match the
  rhythm of the items it separates.

## Gotchas

* A Divider as the first or last child of a container separates a group from nothing, and reads as
  a container border.
* Two Dividers in a row with nothing between them usually means a conditionally hidden group that
  should be hiding its Divider too.
* A rule shows that there is a boundary, not what is on either side of it. If the groups need
  naming, they need headings.
* Divider is decorative (A1) and is never announced, so a grouping that must reach assistive
  technology needs marking in the parent's structure too.
* An inserted Divider must be set to fill unless the slot it lands in stretches its children. This
  one is Figma-side only, and lives in the component set description too, where it is visible at
  the point of insertion rather than only to someone who opens this document.
