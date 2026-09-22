---
name: crd-update
description: Edit an existing CRD — amend a shipped component, convert a legacy CRD to the current format on touch, reconcile a CRD against what was actually built in Figma or code, and maintain front matter, lifecycle and related_crds. Use whenever a CRD file needs changing for any reason other than writing a brand-new one. Not for reviewing a draft (crd-review) or for reading a CRD to implement against it (crd-apply).
---

<!-- ADAPT ME. Placeholders in <angle brackets> are the things that differ per
design system — see the README for the full list. § 5 Ship is one system's
mechanics: rewrite it for yours rather than find-and-replacing it. -->

# CRD Update — amend, convert, reconcile

Three jobs, one skill, because they interleave: an amendment to a legacy CRD triggers a
conversion, and a build triggers a reconciliation that may close amendment statements.

**Boundary.** `crd-apply` locates and applies a CRD during implementation.
`crd-review` critiques a draft pre-implementation. **This skill is the only
thing that writes to an existing CRD.**

## 0. Ground truth — read before acting

CRDs live in `<crd path>`, **one flat folder, no subfolders**. Reference documents sit alongside
them and are the canonical source for anything this skill does not spell out. Read the relevant
one rather than working from memory:

| Document | Owns |
|---|---|
| `crd-how-to.md` | Section-by-section conventions, versioning, the amendment lifecycle, `[open]` markers, the ready-for-dev gate |
| `crd-conversion-map.md` | The legacy → current map: sections, front matter keys, filenames, name-matching hazards |
| `crd-template.md` | The target shape |

> `crd-conversion-map.md` is not included in this share, because it is entirely a function of what
> your *old* documents looked like. Write your own the first time you convert one: a section-by-
> section map, a front-matter key map, the filename rule and its exceptions, and — most
> valuable — the name-matching hazards, the cases where the old document's name for a component
> is not the name the code or the CRD folder uses.

Do not restate their content in a CRD, a commit message or this skill. One home per fact.

**Format detection**: `format: 2` in front matter means current. Absent means legacy. Both are
readable — interpret prose, do not parse. Always **write** to the current shape.

**CRD means Component *Reference* Document.** Not Requirements — most of it is consumed
after requirements are settled.

## 1. Decide which job you are doing

**Whichever job it is, the rule in § 3 applies: classify every difference, then confirm
before editing.** It is filed under reconciliation because that is where it bites hardest, but
conversion surfaces Figma-vs-code divergences just as often, and an amendment can invalidate a
statement nobody meant to touch.

**Amendment** — new or changed requirements for a component. Follow `crd-how-to.md`
§ Amending a shipped component: a transient block placed after the `# CRD: <Name>` title and
its horizontal rule, immediately above `# 1 · Requirements`, statements as `[open]`
with IDs reused where they replace and new where they add, status flipped to `in_progress`.
The block is deleted when design closes it; it is not a permanent record.

**Conversion** — a legacy CRD reaching the substance gate. Follow `crd-conversion-map.md`.

**Reconciliation** — the component was built or changed and the CRD has gone stale. § 3.

### The substance gate

A legacy CRD converts **on touch, never in bulk**. A real amendment triggers it. A typo, a link
fix or a changelog line does not. When conversion is triggered:

* **Two commits.** Conversion first, amendment second — so the amendment stays reviewable
  instead of hiding inside a wall of moved sections.
* **`format:` and `version:` are orthogonal.** Conversion sets `format: 2` and leaves
  `version:` alone; the requirements did not change. The amendment then bumps `version:`
  on its own terms.
* **Never invent.** Content with no source in the old document is either derived from a real
  artefact (the Figma component set, the shipped code) or left empty with its status flag honest.
  Empty is information. Guessed content reads as approved requirement.
* **Anatomy is load-bearing** and usually has no source in a legacy document. Derive it from the
  build where one exists. Where neither build nor design exists, conversion escalates into an
  authoring interview with the designer — that is a legitimate path, not a blocker.
* **Strip the editor dialect.** If your legacy CRDs were exported from a documentation tool, they
  will carry its markup — inline-styled HTML `<table>` blocks, escaped underscores, non-standard
  horizontal rules. Conversion removes it. (Preserving it only makes sense while that tool still
  round-trips the files. Once it is retired, preserving it is just carrying dead formatting.)
* Cross-component content — token wiring, global a11y standards, naming conventions, theming
  rules — does not belong in a CRD at all. Strip it rather than carrying it across.
* **Never drop an open question silently.** Carry every one across. If an amendment subsumes a
  question, the amendment says which one and why. A question that disappears looks answered.
* **Accessibility: nothing is deleted on conversion.** The boundary — what belongs in a CRD
  and what belongs to `<accessibility agent>` — is owned by `crd-how-to.md`
  § What accessibility belongs in a CRD. Read it; do not re-derive it. The conversion rule:
  component-specific statements get `A` IDs, everything else drops into the labelled
  "carried from v1, not verified" block below them. Only ID'd statements are the contract.
* **Never strip accessibility content by heading match.** Read what is underneath first. A legacy
  heading says nothing about its content, and this is the exact failure that put
  `brand_validation: complete` on 22 CRDs whose contrast sections held only placeholders.
  Deleting a real requirement leaves no trace in the CRD, and nobody notices until a
  component ships without it.
* **Distrust derived front-matter values.** A status flag inherited from a legacy document or set
  by a migration script is a claim, not a fact — check it against the actual content before
  letting it stand. See the 22-CRD case above: the migration matched a *heading* while the content
  underneath was template placeholder.
* **Deriving contrast pairings from code: check which selector uses which token.** Do not infer
  from the token's name. A token whose name reads like link text can turn out to be the
  current-page colour. Where a child component supplies the colour, say so and point at that
  component's CRD instead of duplicating its pairings.

## 2. Amendment specifics

Scope discipline during design, from `crd-how-to.md`:

* Statements the amendment names get built.
* Adjustments forced by the change (padding, optical corrections) are made and reported in
  chat, never written into the CRD — they live in Figma and the tokens.
* If the change makes an existing statement false, correct it with a decision entry **and
  say so**. A "small" amendment invalidating several statements probably wanted to be a
  bigger one.
* Anything neither named nor forced is a future amendment, not this session's work.

Uncertainties split two ways and never appear in both: a provisional statement gets an
`[open]` marker inline; a genuine decision needing a named owner and a deadline goes in the
Open questions table.

**Owner convention.** A question with no owner is not a question. Use a person where one is
known, a role (`Design`, `Dev`) where the work is clearly assigned but the individual is not,
and `[needs owner]` only when it genuinely has none — then say so in your report rather than
leaving it buried in the table.

**A question the build has already answered is not open.** Close it: delete the row and log the
answer as a dated entry under Design decisions or Implementation decisions. Only leave it open
if the shipped answer is itself under review.

## 3. Reconciliation specifics

**Gather both sides programmatically, never from memory.**

* **Figma**: inspect the component set — variant axes and options, props, bound tokens and
  resolved values, sizes per breakpoint, forced settings on consumed components, Dev Mode
  annotations.
* **Code**: props interface and types, design tokens consumed, behaviours (focus, keyboard,
  responsive).

**Classify every difference, then confirm before editing:**

* **Build drift** — the build deviates, the CRD is right. The fix belongs in the build.
  Flag it; do not paper over it.
* **Design evolution** — a deliberate decision. Update the CRD, log the rationale in
  Design decisions or Implementation decisions.
* **CRD gap** — built but never spec'd. Add it.

Present the diff concerns-first, quoting exact lines, and get the CRD owner's call on anything
ambiguous **before** touching the file. Never silently make the CRD ratify a mistake.

## 4. Front matter is machine-owned

Everything in the block is maintained by skills, not by hand. This skill writes:

* `status:` keys as work closes them, and `last_updated:` on every edit
* `version:` — patch for corrections, minor for new capability, **major when an amendment
  folds in**
* `lifecycle:` — flip `backlog` → `active` when work starts; set `built` when the component
  ships and the `storybook:` link is written
* `figma:` and `storybook:` when those links become real
* `related_crds:` — **maintained in both directions.** A reciprocal back-link is front-matter
  maintenance, not a requirements change: update the other CRD's `last_updated`, and do **not**
  bump its `version`. Writing one component should not nudge the version of every component it
  cites. Adding a dependency adds this
  component to the other CRD's "used in"; adding a precedent adds this one as a precedent
  there, because consistency is symmetric. Report which files you touched.

### Interim implementations

When a component ships as an **interim** implementation ahead of a design-system one, this skill
records it. Two things change together:

* `lifecycle: active` → `interim`
* the `interim:` block gains `location` (where the implementation lives) and `status: live`

When the design system later supersedes it, `lifecycle` moves `interim` → `built`, the
`storybook:` link is written, and the interim block's `status` becomes `in-deprecation` with a
`retire_by` date once one is agreed. **The CRD does not go to `deprecated`** — the component is
not retired, only that implementation is. Vocabulary and transitions: `crd-how-to.md` § Lifecycle.

**If you are a build skill, do not write this front matter yourself.** Hand over the facts —
component name, that an interim implementation now exists, and where it lives — and let this
skill make the change. Front matter has one writer for the same reason a CRD has one home per
fact: two writers produce two conventions, and the drift is invisible until something is built
from the wrong side of it.

**Three transitions are human-only**, because they are decisions rather than consequences:
`review: complete`, `requested` → `backlog`, and `built` → `deprecated`. Never set them
yourself. Ask.

**Never store ready-for-dev.** It is computed — see `crd-how-to.md` § The ready-for-dev gate.
Storing a second copy of a computable fact is how drift starts.

## 5. Ship — branch, PR, work item

<!-- REWRITE THIS SECTION FOR YOUR SETUP.

     The specifics below — branch names, a commit-message hook, a work-item
     linking check — are one system's. What transfers is the shape:

     1. CRD changes ship through review, like code. Documentation that can be
        pushed straight to the integration branch stops being reviewed within
        about a month.
     2. Stage the CRD file explicitly. Never `git add -A` — an agent editing
        docs should not be able to sweep an unrelated working-tree change into
        a docs PR.
     3. Write down the awkward mechanics of YOUR setup here, once, including
        the failure modes and what they actually mean. The notes below about a
        denied push and an unnecessary `--no-verify` are the genuinely useful
        kind: each one is a wrong diagnosis someone already paid for. -->

Every change goes through a pull request. Do not assume documentation is exempt: an exemption that
once existed may have been removed, and the failure mode is a confusing push rejection rather than
a clear message.

```
git -C <repo> checkout <default branch> && git -C <repo> pull --ff-only
git -C <repo> checkout -b docs/<name>-crd-<change>
# edit the CRD
git -C <repo> add <crd path>/<Name>_CRD.md   # CRD file(s) ONLY
git -C <repo> commit -m "docs(crd): …"       # scope is mandatory
git -C <repo> push -u origin docs/<name>-crd-<change>
```

Then raise the PR with a **linked work item** in the description, in whatever form your tracker
expects. CI fails the PR without one. `<ticket skill>` creates the work item.

**Put the work-item reference in the commit message too, not just the PR body.** A tracker that
watches the repo will then create the commit and pull-request links on the work item by itself,
and there is no reverse link to write by hand.

* **`--no-verify` is not needed** — verify this for yourself once, then record the answer here.
  The commit-message hook enforces `type(scope): description` with the **scope mandatory**, and
  rejects a message identical to the previous commit's.
* **Stage CRD files explicitly.** Never `git add -A`.
* **If `git push` is denied by the agent harness, that is command SHAPE, not a missing
  permission.** `cd X && git push …` fails because it does not start with `git push`;
  `git push … | tail` fails because it is compound. **Retry bare as
  `git -C <absolute path> push …` before concluding anything** — do not hand the command over.
* If a change spans a CRD **and** anything else, they can share one PR, but say so in the
  description — a reviewer reading "docs(crd)" will not expect a skill change inside it.
* Mirror substantive changes on whatever record lives beside the component in Figma — a context
  card, a changelog — so both records cite each other.

## When to use

* "Add X to the Y CRD" / "the CRD needs amending for …"
* Post-build wrap-up after a component design or theming skill
* A component changed in code or Figma and the CRD has gone stale
* A legacy CRD needs bringing to the current format because something substantive is changing
  in it
