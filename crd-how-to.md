# How to work with CRDs

A guide for humans. Agents get their own instructions via skills & the notes embedded in the template; this is for you.

## What a CRD is

**CRD stands for Component *Reference* Document.** It was Component *Requirements* Document until September 2026, & the old name was doing quiet damage: only the first of its sections is requirements, & calling the whole thing a requirements document told people it was finished the moment development started. It is a reference — consulted for the life of the component, not filed away at handover.

A Component Reference Document is a living document that travels with a component through its whole life: requirements before design, specifications after design, implementation decisions before & during dev, and changes after ship. It is not a spec dump & it is not documentation. Its job is the delta: everything that Figma, the codebase & the agent rules cannot tell you. Behaviour, accessibility intent, rules, edge cases, decisions & the reasons behind them.

Where Figma, the code, Storybook or the agent rules express something fully, link to it rather than restating it. Where they express it only partly — a property name is in Figma, but the intent behind it is not — the CRD carries what is missing, & may repeat the name it hangs off. One home per fact; the exception is a key you need to hang the missing part on.

## Related CRDs

`related_crds` is a context-loading instruction, not a relationship map. A CRD earns a place there if an agent working on this component needs to read it to do the job correctly. Two reasons:

**Dependency** — children named in the anatomy, & parents that enforce something on this component (Modal dictating its footer button sizes). An agent *must* read these; missing one breaks the build.

**Precedent** — structurally similar components this one should stay consistent with. An agent *should* read these before inventing an approach; missing one produces drift rather than failure.

**Related CRDs are maintained in both directions.** Adding a dependency to this CRD means that component's "used in" gains this one, & its front matter too if it is genuinely constrained by this component. Adding a precedent means that component's front matter gains this one as a precedent, because consistency is symmetric. The asymmetry matters: a child does not need to load its parent to be built, so mirroring every dependency both ways would inflate context for nothing. The skills do this automatically & report which files they touched.

The test: would an agent build this wrong without reading it? If no, it belongs in Integration prose, not the front matter. The "used in" list can be long & non-exhaustive precisely because nobody loads it.

## Lifecycle

`lifecycle` is the CRD's state, at a glance:

* **requested** — proposed, not committed to
* **backlog** — accepted, nobody has started
* **active** — being worked on now
* **interim** — shipped as an interim implementation, ahead of a design-system component existing. Think
  of it as backlog with a head start: something exists and is usable, but no designer has
  considered it yet
* **built** — shipped in the design system proper; the CRD now serves its post-ship consumers
* **deprecated** — the component is retired, the CRD kept for history
* **rejected** — proposed & declined. The CRD stays, so the same request cannot arrive again
  without its reasons attached

One track, one CRD:

```
requested -> backlog -> active -> built
                    \-> interim -/
```

`interim` sits **before** `active`, not after it. An interim implementation is un-designed by
definition, so nothing has reached `active` yet; when a designer picks it up to harden it, that is
exactly when it becomes `active`, and `built` when the work lands in the library. A component that never
needs an interim simply goes `backlog` → `active` → `built`.

`active` therefore means **a designer is designing it** — precisely what an interim component
lacks. Keeping the field that strict is what lets "has this been designed?" be answered from the
lifecycle alone.

It is one document throughout — there is no second CRD and no hand-off between records.

**`interim` describes the CRD, not the winding-down of the code.** Once the hardened component
lands in the library the CRD is `built`; it is the *implementation* that retires, and that lives in its
own block:

```
interim:
  location:            # where the interim implementation lives
  status:              # live | in-deprecation
  retire_by:           # YYYY-MM-DD, once a deadline is agreed
```

The deadline sits there rather than in `lifecycle` because it governs the code, not the
document — and a CI rule should not have to read a document's state to decide whether to block
a build.

It replaces the old folder taxonomy. Every CRD lives in one directory, because a path is a second home for a fact & moving a file breaks every link pointing at it. The folders never worked as a workflow anyway: no CRD was ever moved between them, `archived` stayed empty, & one amendment ended up filed in two folders at once.

Nothing here duplicates `status`. Anything you might reach for — in design, ready for dev — is already answerable from the status map & the ready-for-dev condition, & storing a second copy of a computable fact is how drift starts.

## Who reads what

Every section states its audience at the top. In short:

| Section | Written | By | Consumed by |
|---|---|---|---|
| Amendment | While a change is in flight | Designer/PM | Everyone: it flags the contract is under revision |
| 1 Requirements | Pre-design | Designer | Everyone: it is the contract |
| 2 Design specifications | Post-design | Designer | Devs & build agents |
| 3 Implementation | Pre-dev & during | Developer | Build agents, future maintainers |
| 4 Documentation guidance | Post-design & post-dev | Designer & dev | Docs generation & usage agents |
| 5 Migration | Only when consumers must act | Whoever changes it | Consumers of the component |

Section 4 exists because the CRD stays relevant after the component ships: docs generation, & the people deciding whether to reach for this component at all, consume it long after the build is done. If you are building the component, skip it; do not delete it. The same goes for any post-ship section your system adds — see § Adding your own sections.

## The requirements contract

Only the ID'd bullets in section 1 make up the contract: F1, S1, B1 & so on. If a line has an ID, it is a requirement; if it does not, it is vocabulary, framing or routing (Purpose, Scope, Anatomy, Ruled out, Open questions, Notes).

That means the contract is not just the subsection called Functional requirements. States, Behaviour, Content, Accessibility & Integration statements are requirements too; those subsections exist because that is how people think & review, not because their contents are softer.

Everything else follows two rules:

**Atomic & testable.** Each bullet states exactly one requirement, phrased so it can be verified true or false. A bullet that could be half-satisfied is two bullets — "a button has a leading or trailing icon, never both, and icons render at the size's icon token" can pass & fail at once, so the audit cannot say which. "A button has a leading OR a trailing icon, never both" passes; "icons should feel balanced" does not. Statements that cannot be machine-tested (label voice rules, mostly the Content subsection) carry a `[review]` prefix: they are verified in the review & audit pass rather than by tests.

**Numbered.** Per-concern IDs: F for functional, S for states, B for behaviour, C for content, A for accessibility, I for integration. The IDs exist so each requirement can be verified individually: the build maps tests to them & cites the ID in the test name, so a failing test points straight at the statement that demanded it. If it is not written as a statement with an ID, it will not be verified. How the build does that verification is the pipeline's business, not the CRD's — nothing it produces gets written back into this document.

This replaces the definition-of-done list. The DoD's discipline — an enumerable list, one test per line, each provably done — survives intact; the enumerable list is now section 1 itself, so there is exactly one copy of the facts & the checklist cannot drift from the requirements it checks. Pipeline mechanics that used to live in the DoD (`storybook:` set after build) belong in the build skill, written once for every component.

## Anatomy & Ruled out

Two vocabulary sections that make the rest of the document work.

**Anatomy** names the parts: the named regions of the component, the things you can point at in a design (container, label, leading icon, spinner). Some are sub-components, some are plain elements; Anatomy names both without distinguishing. Properties are not parts — `size` is something you set, `container` is something you point at. Everything hangs off these nouns: requirement statements, token paths, Figma layers, prop names, decisions. Write statements using anatomy terms & no others; if you need a word that is not in the anatomy, the anatomy is incomplete.

**Ruled out** records considered-&-rejected design space ("a third size: no evidenced need"), one line each with the why or a link to the decision. It is not out-of-scope, which routes to other components; its job is preventing re-litigation, & it matters most for agents doing best-practice research, which will otherwise cheerfully propose the option you rejected six months ago. When a decision permanently closes a door, add the line here in the same commit.

## Provisional statements & the [open] marker

Pre-design, the CRD is partly a design brief, and briefs are allowed to contain hunches. "Consider stacking buttons vertically on mobile" is a legitimate prompt for design exploration. The rule is that it must be visibly provisional:

```
[open] B4: Consider stacking buttons vertically on mobile
```

Design closes every marker with one of three exits:

1. **Promote.** It becomes a testable statement: "B4: Buttons stack vertically below the mobile breakpoint." Log it in Design decisions.
2. **Move.** It turned out to be advice, not a requirement. It goes to Documentation guidance as a usage pattern.
3. **Cut.** It did not survive contact with the design. Log it in Design decisions with the why.

Bigger uncertainties that need a named owner & an actual decision go in the Open questions table instead, each with its deadline in the By column ("before auth screens are designed"): timing is often the real content of a question. Do not put the same thing in both places.

## The ready-for-dev gate

"Ready for dev" is a condition, not a status you set. It is true when all of these are:

* `requirements: complete` & `design_specs: complete` in the front matter
* `review: complete` — the designer & a developer have jointly reviewed the CRD against the Figma file
* Zero [open] markers anywhere in section 1
* Every Open questions row closed or explicitly marked "deferred: [reason]"

Documentation guidance is not a dev blocker; it serves later consumers, so it does not hold the gate — & nor does any post-ship section your system has added.

It is computed rather than stored because a stored flag can be true in the front matter & false in reality the moment anything upstream changes. Anyone — human or agent — can check the condition in seconds, & a build agent verifies it before starting rather than trusting a flag someone set optimistically.

The joint review is not a formality, & it is the one part of the condition nothing else can infer, which is why it gets its own field. It is where CRD-vs-Figma conflicts get caught before they cost a developer confidence mid-build. Check that the design covers every statement, that every dual option (link or button, badge or chip) is impossible to miss in both places, & that nothing in the CRD contradicts the file.

If a build agent finds an [open] marker or an open question, it stops & flags rather than guessing. That is deliberate. Guessing is how components get built wrong quietly.

## When Figma & the CRD disagree

Precedence, in order:

1. Figma for visual specification
2. The CRD for behaviour, accessibility, rules & requirements
3. Agent rules for code conventions & implementation standards
4. Storybook & code for the built API

If they conflict on the same fact, do not silently pick one. Flag it. A conflict means one of them is stale, & picking the wrong one hides the staleness.

## Developers: your sections

Two habits, both in section 3.

**Implementation decisions, before you build.** If you research best practices, compare how other libraries handle something, or decide on a pattern ("render status as a child of the indicator, not an overlap prop"), capture it here, dated, one line plus a short why. Putting it only in your prompt means the decision shapes one build & then evaporates. Putting it here means it shapes every rebuild, refactor & audit of this component, & the next person (or agent) does not re-litigate it from scratch. Two habits within the habit: record reversals explicitly ("reverses the earlier X decision"), & cite precedents from other CRDs by name ("the Divider lesson", "the MenuRow rule"). Named precedents build case law across the library: one decision, cited three times, becomes a convention no one has to restate.

**Dev notes & deviations, during the build.** Limitations you hit, quirks, performance notes, & anything you did differently from the design or the requirements. Deviations get flagged to the designer, not just recorded; the CRD is not a place to bury a disagreement.

## Designers: your sections

Section 1 is yours before design; sections 2 & 4 are yours after, along with any post-ship section your system has added. The one that needs discipline is **Design decisions**. The bar for logging: could a future person or agent plausibly propose the opposite? If yes, log it with the why. If no — you reworded a statement, you closed a question — it is a commit message, not a decision. And if a new entry fully supersedes an old one, say so & delete the old one; never leave two entries where one is now wrong. Every change from the original requirements gets a dated line. If the design changed a requirement, update the statement in section 1 in the same commit as the log entry. A decision log that disagrees with the requirements above it is worse than no log.

For section 2, the Properties table lists every property, so an absent row means the property does not exist. Only the description column does real work: intent, cross-property rules, why a property exists — the things MCP cannot surface. Leave it empty where Figma captures the property fully; an empty description reads as "nothing to add", which is information. There is no code properties table & no Figma-to-code mapping on purpose: the development team owns the API, & the built API documents itself in Storybook & the code.

## Section by section

What goes in each section, & who consumes it. The template is a bare skeleton; this is the guidance that used to sit inside it.

**Purpose** — two or three lines: the problem this solves & its primary job. No feature lists.

**Scope** — what this is & is not. Every out-of-scope line points at the correct alternative component, so neither humans nor agents reach for this one wrongly.

**Anatomy** — the named parts (see above). One line each; mark optional parts.

**Functional requirements (F)** — capabilities & rules. Edge cases are requirements & live here, not in a later section. State rules so they are impossible to miss ("F4: a button has a leading OR a trailing icon, never both").

**States (S)** — every state, one line each: interactive (hover, focus, pressed) & application (disabled, loading, error). The state vocabulary is shared across the CRD, Figma, tokens & code — one name per state, everywhere.

**Behaviour (B)** — interactions, keyboard behaviour, responsive behaviour, animation & motion. Figma cannot express this, so be complete here rather than thorough about visual description.

**Content (C)** — label voice, length, casing, what copy must & must not say. Most of these carry `[review]`.

**Accessibility (A)** — component-specific only; see § What accessibility belongs in a CRD. Include the ARIA attributes & values this component sets, what it announces & when, its focus order & focus management, & anything unusual to it. Do not restate global standards.

**Integration (I)** — composition rules this component enforces on its children, plus structural rules. Ownership: if a parent enforces something on a child (sizes, placement), that lives in the PARENT's CRD. Children & Used in are linked lists, not requirements.

**Ruled out** — rejected design space (see above).

**Open questions** — owner, By & status per row. Small uncertainties stay inline as `[open]` markers instead.

**Notes** — relevant context that is not a requirement: brand variability expectations, constraints from outside the component. Leave empty rather than pasting boilerplate.

**2 · Design specifications** — the properties table & design decisions (see above). Written post-design.

**3 · Implementation** — implementation decisions before you build, dev notes & deviations during (see above).

**4 · Documentation guidance** — when to use, when not to (pointing at the alternative each time), common patterns, & gotchas. This is where `[open]` statements land when they turn out to be advice rather than requirements. Consumed by docs generation & the designer-facing usage agent.

**5 · Migration** — only while a change requires consumer action.

## Adding your own sections

Sections 1 to 4 are the portable core: the contract, what design decided, what dev decided, and how to use the thing. Most systems need at least one more, and the place for them is between 3 and 4 — after the build detail, before the usage advice.

Two that earn their place in a lot of systems:

* **Brand or theme validation** — the token pairings that need contrast checking, split by when they *can* be checked: at brand creation for opaque fills, at placement for transparent ones that need the parent surface to resolve. Plus the exemptions, & anything the validation tooling needs to know. Consumed by whatever sets up brands, not by the component build.
* **CMS or page-builder behaviour** — what a self-serve, non-designer user gets when they place this component: layout constraints, the subset of properties exposed to them, & the guardrails that stop them producing a broken or inaccessible configuration. Consumed by the page-builder work, not by the component build.

The Divider example alongside this guide carries both, so you can see the shape.

Whatever you add, three rules keep it from rotting:

1. **Give it an audience line** in the Who reads what table. A section nobody is named as reading gets written once & never updated.
2. **Add a matching key to `status:`** in the front matter, so "not started" is sayable & an empty section is honest rather than ambiguous.
3. **Decide whether it gates dev.** Post-ship sections should not — say so explicitly, or people will hold builds waiting for a section that serves someone else entirely.

Resist adding a section for something that already has a home. If Figma, the code, Storybook or your agent rules express it fully, the CRD links to it. One home per fact is the whole discipline; a new section is the easiest place to break it.

## What accessibility belongs in a CRD

An accessibility statement earns its place only if it is true of **this component and no other**. Everything else already has a home: your global accessibility rules — for us, a `.claude/agents/accessibility-expert.md` agent loaded on every component build & review. This section assumes you have one. If you do not, write it before you write any CRD, or every CRD will re-state the same twenty rules & each copy will drift.

**The test.** Would this sentence still be true pasted into a different component's CRD? If yes, it is not yours. If no, keep it & give it an `A` ID.

**Never write these.** All are covered by the accessibility agent:

* Semantic HTML — element choice, link-vs-button, heading hierarchy, list markup
* ARIA mechanics — when to reach for `aria-label`, `describedby`, `labelledby`, `live`, `expanded`, `current`; prefer-native-HTML
* Contrast ratios — 4.5:1 text, 3:1 large text, UI components, non-text & focus indicators
* The disabled-state contrast exemption
* Keyboard conventions — that tab order follows DOM order, Enter/Space activating buttons, Escape closing dialogs, arrows within groups
* Focus indicators — visibility, 2px, 3:1, `:focus-visible` over `:focus`
* Screen reader basics — alt text, label association, announcing dynamic content
* RTL — logical properties, `:dir(rtl)` flips, directional icons
* Reduced motion — `prefers-reduced-motion`
* Touch targets — the 44x44 minimum & the pseudo-element technique
* Modal behaviour — focus trap, focus return, `aria-hidden` on page content
* Loading states — `aria-busy` & `aria-live`
* Form field wiring — `aria-required`, `aria-invalid`, error association
* WCAG criterion numbering — cite from the agent's reference table

**Do write these.** Each names something specific to this component:

* The ARIA attribute it sets & the value it sets — `aria-label="Breadcrumb"`, `aria-current="page"`
* What is announced, when, & how politely. An `aria-live` politeness choice is a design decision
* Which parts are deliberately decorative & excluded from the accessibility tree
* **Focus order within the component**, where it is not simply DOM order, or where a part is deliberately not focusable. Breadcrumb's current page is skipped, & that is a requirement rather than an accident
* **Focus management on state change** — where focus goes after a dismiss, delete, expand, collapse, close or async update. The agent covers the modal case only; every other component's answer is a design decision
* Keyboard behaviour beyond platform convention
* Accessibility-driven design decisions — they start as an `A` statement & end up in Design decisions

**No `A` statement may be a principle alone.** Every one names an attribute, a value, an announcement or a behaviour. "Must be accessible" & "must meet WCAG AA" are not requirements; they are true of everything.

**Failing the test does not promote it to the agent.** Three outcomes: it is already covered, so it goes (the usual case); it is a genuine gap, so raise it against the agent rather than writing it here; or it is component-specific but written too vaguely, so rewrite it & keep it. The same sentence appearing in three or more CRDs is a prompt to go looking for a missing global rule, not grounds to promote one.

**Tidying an existing CRD: nothing is deleted on suspicion.** A statement that looks like it fails the test is not removed on the spot. It drops below the ID'd ones into a labelled block:

> _Not verified as component-specific. Check against the global accessibility rules & delete once confirmed covered._

Only ID'd statements are the contract, so the block cannot be mistaken for one, & nothing is lost while it waits to be checked. **Read the content under every accessibility heading before moving anything — never match on the heading alone.** A heading tells you nothing about what sits under it, & real requirements hide under generic-sounding ones: "Focus management" is the usual offender, & much of what it covers is genuinely component-specific.

## Front matter is not yours

Everything in the front matter block is maintained by the skills, & not by hand. `lifecycle`, `status`, `version`, `format`, `last_updated`, `figma`, `storybook`, `related_crds`, `id` & `name` are all either derivable from the work or need to change in more than one file at once — `related_crds` is maintained in both directions, a status change moves `lifecycle` from backlog to active, closing a section bumps the version. A human editing one of them by hand updates one copy of a fact that has two, & the drift is invisible until something is built from the wrong side of it.

So the writing happens through Claude Code, & you read the result rather than typing it. In practice that means both windows open: Claude Code doing the editing, & the CRD in preview beside it so you can see each change land in context. An occasional hand tweak to a line of prose is fine. Front matter, never.

The exceptions are all one exception: a decision nothing can infer. `review: complete` is set by
a human, because the joint designer/dev review is the one condition nothing can derive — a skill
claiming it would be claiming a conversation happened. `requested` to `backlog`, `requested` to
`rejected`, & `built` to `deprecated` are human for the same reason.

Everything else is a consequence of work that already happened, & consequences are the machine's
job — including `backlog` to `interim` when an interim implementation ships, `backlog` or
`interim` to `active` when a designer starts work on it, & `active` to `built` when the component
lands in the library.

## Versioning

The version in the front matter tracks this document's maturity, at a glance. The skills apply this scheme; it is here so you can read a version, not so you can set one. It has no relationship to the components package version: the package releases the whole library at once, so a per-component CRD version could not mirror it even if you wanted it to.

* **Patch** — tweaks within a section: corrections, clarifications, a decision entry, a closed open question, the code link added after build.
* **Minor** — a section reaches complete.
* **1.0.0** — the CRD is complete & the contract is fixed. That is the gate, not the ship date; dev happening afterwards is a patch.
* **Major** — an amendment completes. Each major version is one generation of the component's requirements contract, so 2.0.0 is the contract after the first amendment, & the code link tells you which generation is built.

## Amending a shipped component

Three months after the card ships, it needs significant new functionality. Do not splatter the new requirements through the old contract, & do not keep the superseded ones for reference. Amend first, then rewrite.

**Write the amendment.** A section at the top of the document, above section 1: a sentence or two of context (what is changing & why now), then the new & changed requirements as provisional `[open]` statements, reusing existing IDs where they replace a statement, new IDs where they add one, deletions noted. Nothing more structured than that — it exists for one design cycle & then it goes. It sits at the top because anyone reading the contract needs to know it is under revision before they act on it. Flip the status to in_progress.

**Scope during design.** Statements the amendment names get built. Design adjustments forced by the change (padding, spacing, optical corrections) get made & reported in chat, never written into the CRD: they live in Figma & the tokens. If accommodating the change makes an existing contract statement false, that statement gets corrected with a decision entry, & you should hear about it — a "small" amendment invalidating several statements probably wanted to be a bigger amendment. Anything neither named nor forced by the change is a suggestion for a future amendment, not this session's work.

**Design closes it.** Same as any `[open]` marker: every statement is promoted, moved or cut, & logged in Design decisions. The why-now from the amendment's context line lands here too, which is where it lives permanently.

**Then fold it in.** Statements are rewritten, added, or deleted outright in section 1. Requirements that no longer apply are removed, not annotated as superseded: git holds the old text, the decision logs hold the why, & a contract carrying its own history becomes unreadable fast. Rejected options join Ruled out.

**Then delete the amendment section**, return the status to done, & bump the major version. If the change requires consumer action, write the Migration section as you fold the amendment in.

There are no separate amendment CRDs. Git carries the diff, the logs carry the why, the version signals movement, & Migration carries consumer instructions when they are needed. A parallel document would be a fifth copy of facts that already have four homes.

## Keeping it alive

CRDs are updated through Claude Code, alongside the work, in the same commit as the thing that changed. The logs are dated one-liners precisely so that updating is an addition, not a rewrite. If updating a section feels expensive, that is a signal the section is duplicating something with a home elsewhere.

If you ever see an [open] marker in section 1 of a completed CRD with no amendment in flight, someone changed a requirement without closing the thought: treat it as a flag, not furniture.

## Quick reference

* One home per fact; link, never duplicate
* Every contract statement: atomic, testable, ID'd; not machine-testable = [review]
* One test (or more) per non-[review] ID; test names cite the ID
* Statements use anatomy nouns & no others
* Rejected options go in Ruled out, with the why
* Provisional = [open] prefix; design closes every one
* Ready for dev is computed: requirements + design_specs + review complete, no markers, no open questions
* Conflicts: stop & flag, never silently pick
* Log a decision only if someone could plausibly propose the opposite; supersede & delete, never stack
* Version: patch = tweaks, minor = section complete, 1.0.0 = contract fixed, major = amendment closed
* Changing a shipped component: amendment at the top, then fold in & delete it
* Verification output stays in the pipeline, never in the CRD
