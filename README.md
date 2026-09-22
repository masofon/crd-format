# CRDs — Component Reference Documents

A format, a set of conventions, and two Claude Code skills for writing and maintaining the
document that travels with a design system component through its whole life.

**CRD stands for Component *Reference* Document.** It was *Requirements* for a long time, and the
old name was doing quiet damage: only the first section is requirements, and calling the whole
thing a requirements document told people it was finished the moment development started. It is a
reference — consulted for the life of the component, not filed away at handover.

Its job is **the delta**: everything that Figma, the codebase, Storybook and your agent rules
cannot tell you. Behaviour, accessibility intent, rules, edge cases, decisions, and the reasons
behind them. Where another source expresses something fully, the CRD links to it. One home per
fact.

This is a working format lifted out of a real design system, with the system-specific parts
genericised. It is not a product and it is not finished — it is what a team actually uses, shared
because people asked for it.

## What's here

| File | What it is |
|---|---|
| `crd-how-to.md` | **Start here.** The conventions, written for humans: the requirements contract, ID'd statements, `[open]` markers, the lifecycle, versioning, the ready-for-dev gate, what accessibility belongs in a CRD and what does not, and how to amend a shipped component |
| `crd-template.md` | The skeleton. Four numbered sections plus a commented Migration section |
| `Divider_CRD.md` | A real, filled-in CRD — see the caveats below |
| `skills/crd-write/SKILL.md` | Claude Code skill: research, interview, and draft a new CRD, or fill a later section of an existing one |
| `skills/crd-update/SKILL.md` | Claude Code skill: amend and reconcile an existing CRD, and maintain its front matter |

Read `crd-how-to.md` first even if you only want the skills. The skills assume it and defer to it
constantly; on its own, a skill reads as a pile of rules with the reasoning removed.

## The shape of it

```
1 · Requirements            pre-design    the contract — ID'd, atomic, testable statements
2 · Design specifications   post-design   what design decided, and why
3 · Implementation          pre/during    what dev decided, and what deviated
4 · Documentation guidance  post-ship     when to use, when not to, gotchas
5 · Migration               only when consumers must act
```

Only the ID'd bullets in section 1 are the contract: `F1`, `S1`, `B1`, `C1`, `A1`, `I1`. If a line
has an ID it is a requirement; if it does not, it is vocabulary, framing or routing. Each one is
atomic and testable, so tests can cite the ID and a failure points straight at the statement that
demanded it. Statements that cannot be machine-tested carry a `[review]` prefix. Provisional ones
carry `[open]`, and design closes every one.

Sections 1 to 4 are the portable core. Most systems need at least one more — see
**§ Adding your own sections** in the how-to.

## Adapting it

### Placeholders

The two skills carry placeholders in `<angle brackets>`. Find and replace:

| Placeholder | What it means |
|---|---|
| `<crd path>` | Where CRDs live, one flat folder — e.g. `docs/crd/` |
| `<component path>` | Where component source lives — e.g. `packages/components/lib/` |
| `<design foundations doc>` | Your file describing Figma file keys, token tiers, the theming model, text styles, spacing scales |
| `<accessibility agent>` | Your global accessibility rules, loaded on every component build and review |
| `<ticket skill>` | Whatever creates a work item in your tracker |
| `<default branch>` | Your integration branch |
| `<repo>` | Path to the repo |

### Blocks to rewrite, not find-and-replace

Three blocks are marked with HTML comments explaining what transfers and what doesn't. The comment
is the instruction; delete it once you've done the work.

- **`skills/crd-write` § House doctrines** — the handful of house positions a new author would otherwise
  get wrong by reasonably assuming the opposite. The list shipped here is one system's, kept only
  so you can see the shape. Replace it wholesale.
- **`skills/crd-write` Phase 3, question 4** — "how does this component take colour?". The three options
  given are one system's theming model. Keep the question, replace the options. Whatever your
  model is, a component whose colour behaviour nobody decided gets built against whichever tokens
  happened to be nearest, and nobody finds out until it's placed somewhere unexpected.
- **`skills/crd-update` § 5 Ship** — branch, PR and work-item mechanics. Rewrite for your setup. What
  transfers: CRD changes ship through review like code; the CRD file is staged explicitly so an
  unrelated working-tree change can't ride along; and the awkward mechanics get written down once,
  in one place, including the failure modes and what they actually mean.

### Not included

- **`crd-review`** and **`crd-apply`** — two sibling skills the others reference. `crd-review`
  critiques a draft; `crd-apply` reads a CRD to implement against it. Referenced by name so the
  boundaries are legible; write your own or delete the references.
- **The global accessibility rules.** The how-to's accessibility boundary assumes you have a
  single document holding every rule that is true of all components — semantic HTML, ARIA
  mechanics, contrast ratios, keyboard conventions, focus indicators, touch targets, reduced
  motion, RTL. If you don't have one, write it before you write any CRD, or every CRD will
  restate the same twenty rules and each copy will drift.

## About the Divider example

`Divider_CRD.md` is a **real CRD from Resin**, Unily's design system — shared verbatim, with only
a Figma file link and a tracker reference redacted. It is deliberately *not* genericised, so you
can see what one of these looks like in service rather than as a specimen.

**It doesn't match the template, and that's the point.**

Divider has six numbered sections where the template has four. The two extra ones are Resin's, and
they're the clearest demonstration in this repo that the format is meant to be cut to fit:

| Section | What it's for | Who reads it |
|---|---|---|
| `4 · Brand validation` | Which token pairings need contrast checking, split by *when* they can be checked — at brand creation for opaque fills, at placement for transparent ones that need the parent surface to resolve. Plus the exemptions, and why each one is deliberate | Resin's Brand Builder and the people setting up a new brand — **not** the component build |
| `5 · CMS behaviour` | What a non-designer gets when they place this component in the CMS: layout constraints, which properties are exposed to them, and the guardrails that stop them producing something broken or inaccessible | The CMS block work and the runtime agent — **not** the component build |

Both exist because Resin is a multi-brand design system behind a CMS product, so a component's
CRD has to answer questions long after the component ships, for people who will never open Figma.
If your system doesn't have those consumers, the sections would be dead weight — which is exactly
why they're not in the template. A system with a performance budget, a localisation review or a
native platform to mirror would add something different.

Two mechanical consequences worth noticing, because they're what "adding a section" actually costs:
its Documentation guidance is numbered **6** rather than 4, and its front matter carries
`brand_validation:` and `cms_behaviour:` keys in the `status:` map. That's the extension working
as intended, not drift. See **§ Adding your own sections** in the how-to for the rules that keep
an added section from rotting.

**Two other things to know:**

- **It names real components** — Context Menu, List — real token paths, and a couple of Resin's
  own skills. Treat them as texture, not as anything you need to map onto your own system.
- **It is a small component on purpose.** A one-part atom with no states, no content and no
  interaction is the clearest way to see what the format does, because almost everything in it is
  *reasoning* rather than specification. Note how much of its value sits in **Ruled out** — nine
  rejected options, each with its why. That section exists to stop the same proposal arriving
  every six months, and it's the one people skip when they adopt the format.

## Installing the skills

The two skills are [Claude Code](https://claude.com/claude-code) skills. Drop them in:

```
.claude/skills/crd-write/SKILL.md
.claude/skills/crd-update/SKILL.md
```

Either in your repo (shared with the team, checked in) or in `~/.claude/skills/` (just you). They
route off the `description` in each one's front matter — "write a CRD for X" reaches `crd-write`,
"the CRD needs amending" reaches `crd-update`.

Both skills expect to read `crd-how-to.md` and `crd-template.md` at `<crd path>` every time they
run. Put those files next to your CRDs, not somewhere clever.

## A note on what makes this work

Not the template. The template is a skeleton and took an afternoon.

What makes it work is the set of rules the skills enforce that nobody enjoys enforcing by hand:
one home per fact; every statement atomic and testable; every rejection recorded with its why;
every decision logged only if someone could plausibly propose the opposite; front matter written
by machine because a human maintaining two copies of a computable fact is how drift starts; and
ready-for-dev *computed* rather than stored, so a flag can never be true in the front matter and
false in reality.

Those are the transferable parts. The section headings are not.

## Contributing

Issues and PRs welcome, particularly:

- **Adaptation notes.** If you've fitted this to a system with a different theming model, token
  architecture or tracker, the interesting part is what didn't fit. That's more useful than a
  clean success story.
- **Corrections.** Places the how-to contradicts itself, or a rule that turns out not to survive
  contact with a different kind of component.

No template or process for raising something — just open an issue.

## Origin

Developed for the [Unily](https://www.unily.com) Resin design system, and shared here with the
system-specific parts genericised. Some of the reasoning still carries the shape of the system it
grew in; the Divider example most of all.

## Licence

[MIT](LICENSE) — use it, change it, ship it, no attribution required.
