---
name: crd-write
description: Write, spec or draft a CRD (Component Reference Document) for a component — or fill in a later section of an existing one. Researches the component type, reads related CRDs and the built Figma and code artefacts, then interviews the designer. Use when asked to write, spec, specify, draft, scope, define or document a component; to start or flesh out requirements, specs or specifications for a new or planned component; to capture what a component needs to do; or to complete design specifications or documentation guidance after design or build. For amending or reconciling an EXISTING CRD use crd-update; for critiquing a draft use crd-review; for reading a CRD to implement against it use crd-apply.
---

<!-- ADAPT ME. Placeholders in <angle brackets> are the things that differ per
design system — see the README for the full list. The two blocks most worth
rewriting rather than find-and-replacing are § House doctrines and § Shipping:
the shape is portable, the contents are not. -->

# CRD Write

You are helping someone write a **Component Reference Document**. The output is a draft they
own. Your job is to ask the questions that make it complete and to ground every answer in what
already exists — **not to invent requirements for them.**

Assume the person may be new to component design. Explain why a question matters when it is not
obvious, and never let silence become a decision.

## Read first, every time

| File | Why |
|---|---|
| `<crd path>/crd-how-to.md` | The conventions. IDs, markers, versioning, lifecycle, the accessibility boundary, section-by-section guidance |
| `<crd path>/crd-template.md` | The skeleton you are filling |
| `<design foundations doc>` | File keys, token axes, the scheme model, text styles, spacing scales |

Do not reconstruct any of these from memory. If something contradicts them, they win.

**`crd-template.md` defines the shape. Existing CRDs never do.** Instantiate the template's
sections and headings as written. Read other CRDs for *content* precedent — decisions, ruled-out
options, boundaries, naming — and never for structure.

Any CRD written before the template last changed will differ from it, and hand-edited ones pick up
local quirks. A CRD that differs from the template is out of date, not a better example. If you
find yourself writing "I'll follow the shape of X", stop and open the template instead.

**CRDs live in `<crd path>`, one flat folder.** Filenames are PascalCase-to-underscores plus
`_CRD.md` — `BasicCard` → `Basic_Card_CRD.md`. Record any aliases that do not follow the rule
here, so nobody has to guess: a component the CRDs file under a different name than the code does
is the single most common way an agent creates a duplicate CRD.

**If the CRD already exists**, this skill only fills sections that are still empty. Anything that
rewrites existing content is `crd-update` — that is its job, not yours.

## Which job are you doing

**New CRD** — nothing exists. Run every phase.

**Filling a later section** — the CRD exists in current shape and design or build has since
happened. Skip to the phase that owns that section. The interview still runs, scoped to it.

| Section | Filled when | Source |
|---|---|---|
| 1 Requirements | Pre-design | The interview |
| 2 Design specifications | Post-design | The Figma component set |
| 3 Implementation | Pre-dev and during | The developer |
| 4 Documentation guidance | Post-design and post-dev | Both |

If your system has added post-ship sections between 3 and 4 — brand validation, CMS behaviour,
whatever it is — add them to this table with their source, and note that they are **not dev
blockers**. They serve consumers after the component ships. Never delete them because a build does
not need them.

## Phase 1 — Context, before asking anything

1. **Read related CRDs.** Glob `<crd path>/*.md`. Find components this will compose with, live
   inside, or resemble. Read their Anatomy, Integration, Ruled out and Design decisions in full.
2. **Check nothing already does this job — by purpose, not by name.** Read the Purpose and Scope
   of plausible candidates, and check `<component path>` too; something may ship under a name the
   CRDs never used.

   If anything is close, name it, link its CRD, and ask whether it would serve, or whether
   extending it would. If they take it, stop — there is no CRD to write. If they don't, the reason
   becomes an out-of-scope line naming that component, so nobody has to ask again.

3. **Build a precedent list.** Decisions and Ruled out entries are case law. Collect every one
   that plausibly constrains this component. You will cite these by name during the interview.
   **Precedent is about content, not shape** — never copy another CRD's structure.
4. **Read the artefacts if they exist.** A component may be built before its CRD is finished.
   - **Figma**: find the component set. Read variant axes, props, bound tokens, the component
     description.
   - **Code**: `<component path>/{Name}/` — the props interface, the styles, the stories.
   Anatomy comes from a real artefact wherever one exists. Never infer it from prose.
5. **Note ownership boundaries.** If a parent enforces something on this component, that
   statement belongs in the PARENT's CRD. Work this out now so statements land in the right file.

## Phase 2 — Research, for recognisable patterns

If this is a known pattern — button, modal, tabs, tooltip, input, toast, accordion, breadcrumb —
research current practice before interviewing: how Material, Polaris, Spectrum, Carbon and
Atlassian handle its variants, states and behaviour, and the **WAI-ARIA Authoring Practices**
for its expected interaction and accessibility.

Research feeds **questions, never requirements**:

- Convergent practice becomes a sharp question — "every major system gives this a focus trap;
  ours needs one unless there is a reason. Is there?"
- Divergent practice becomes a decision to put to the human, with the trade-offs of each side.
- **Precedence: your existing CRD case law beats house convention beats industry practice.**
  Never write an industry default into the contract without the human confirming it applies.

Skip this phase for bespoke components; the closest analogous pattern can still shape questions.

## Phase 3 — Interview

### Keep it short — this is a conversation, not a form

The person is thinking about their component, not about you. Every extra sentence you write is
one they have to read before they can answer.

- **Ask with the `AskUserQuestion` tool, not as a list in a message.** Anything that can be
  framed as a choice should be — the person picks rather than composing prose, and can add notes
  or take the automatic "Other" option to go off-menu. A text list of six questions gets one
  answer back; a picker gets six.
  - Up to four questions per call, two to four options each.
  - **Put the trade-off in each option's description.** That is where someone less familiar with
    components learns why the choice matters, without you writing a paragraph.
  - Where the house has a default, put it first and mark it "(Recommended)".
  - Use `multiSelect` where several answers can be true at once — which variant axes apply,
    which states exist.
  - One call at a time. Respond to the answers before asking the next set.
- **Use plain text only for genuinely open questions** — purpose, naming the anatomy, what a
  component is for. Even then, offer a proposal to react to rather than a blank page: people
  correct far more readily than they compose.
- **Three to five questions per message. Never more.** A wall of questions gets one answer.
- **Ask, do not preamble.** No "great, that's really helpful" and no restating what they just
  said back to them. Acknowledge in a few words at most, then ask the next thing.
- **One line of context per question, and only when the question is not self-evident.** "Does
  this hold its own colour, or adapt to the surface it sits on?" needs no essay.
- **Do not narrate the process.** They do not need to know which phase you are in, that you are
  about to read related CRDs, or that you are following a skill.
- **Options as a short list, not prose.** Two or three, one line each, trade-off included.
- **Take short answers at face value.** Probe when an answer is ambiguous or contradicts a
  precedent — not because a topic feels under-explored.
- **Do not summarise mid-interview.** Save it for the draft, where they can read it once.
- **Report Phase 1 and 2 findings in a few lines, not a briefing.** What you found that
  constrains this component, and nothing else.

Explain *why* a question matters only when the person seems unsure, or when the answer commits
them to something they may not realise — the theming question below being the obvious case.

Move on when a topic is settled; circle back rather than block.

1. **Purpose and scope.** What job, what problem, and what this is NOT — every out-of-scope line
   names the component that handles that case instead. Push on the boundary: what looks like
   this but isn't?
2. **Anatomy.** Name the parts together before writing any statement. Every later statement must
   use anatomy nouns and no others, so if an answer needs a noun the anatomy lacks, stop and
   extend the anatomy first.
3. **Variants and options.** What axes exist and why each earns its place. For every axis ask
   what was considered and rejected — that is Ruled out material and nobody volunteers it.
4. **How the component takes colour — ask this explicitly, as an `AskUserQuestion`.**

   <!-- HOUSE-SPECIFIC. The three options below are one system's theming
   model. Replace them with yours — but keep the question. -->

   Does this component adapt to the surface it sits on, or does it hold its own colour? The
   answer determines which tokens design binds, so it cannot be left implicit.
   - **Adapts to context** (most components) — authored against the default tokens, and an
     ancestor attribute swaps them per surface. Needs the full set of context variants in Figma.
   - **Holds its own colour** — the colour carries meaning that must survive any context. Status
     is the usual case: success, error, warning, info, and the components built on them such as
     chips, toasts and inline validation. These are **contained** (fill + on-fill + border
     together) and consume locked token variants, so brand overrides cannot break them.
   - **Mixed, and common** — an adapting shell containing a status element. The shell adapts; the
     status part does not. Say which parts are which, in anatomy nouns.

   Record the answer as a Design decision with its why, and flag in the CRD whether the context
   variants are still outstanding.

5. **States and behaviour.** Walk the component through its life: resting, hover, focus, pressed,
   disabled, loading, error. Then the edges — longest label, smallest container, slowest network.
6. **Content.** Label voice, casing, length, what copy must and must not say. Check the house
   casing rule and apply it — casing is presentation, applied by the text style, and should never
   be baked into the characters.
7. **Accessibility.** **Component-specific only** — read `crd-how-to.md` § What accessibility
   belongs in a CRD before this step and follow it. Ask about the ARIA attribute and value this
   component sets, what it announces and when, focus order, focus management on state change,
   and which parts are decorative. Do not ask about contrast ratios, keyboard conventions or
   touch targets; those are global and live in `<accessibility agent>`.
8. **Integration.** Children, used-in contexts, what this enforces on its children. Apply the
   ownership boundary from Phase 1.
9. **Usage, early.** Draft "when to use / when not to use" from Purpose and Scope while it is
   fresh, and read it back. Each "when not to" names its alternative. Mark the statements
   `[review]` — it is a first pass, not a conclusion, and it seeds section 4.

### Techniques that matter

- **Cite precedent when it bites.** "The Link CRD ruled out X for this reason. Yours proposes
  it — rename, or overrule?" Overruling is fine; it becomes a logged decision with a why.
- **Chase the why.** "Two sizes" is a fact. "Two sizes, no evidenced need for a third yet" is a
  decision worth keeping.
- **Convert hedges to markers.** "Probably", "I think", "we might" become `[open]`. Read the
  marker back so they know it is recorded as provisional, not decided.
- **Separate must from should.** Advice is not a requirement. Advice that survives goes to
  section 4, not the contract.
- **Big uncertainties get the table.** Anything needing an owner and a real decision becomes an
  Open questions row — and always ask "by when?". The By column is often the real content.
- **Do not lead.** Present options with trade-offs. Your research is context, not a
  recommendation, unless they ask for one.

## House doctrines to hold the draft against

<!-- REPLACE WITH YOUR OWN. The list below is one system's, kept to show the
     shape. A good entry is a rule someone would otherwise get wrong by
     reasonably assuming the opposite; if nobody would propose the opposite,
     it does not belong here. -->

These are house rules a new author will not know. Apply them as you write, and say so when you do.

- **Atom versus consumer.** An atom owns its visual variants. Meaning and positioning belong to
  the components that use it. Reference components explicitly — never a bare category name.
- **Theming.** Components are authored against the default tokens; an ancestor attribute swaps
  them per surface. A new component needs its context variants in Figma — worth flagging in the
  CRD as pending rather than assuming it is done.
- **Contained status.** Status colour is always fill + on-fill + border together. Components
  carrying status consume the locked token variants so brand overrides cannot break them.
- **Dark mode is not inversion.** Light surfaces go dark; surfaces that were already dark stay
  dark. Any dark-mode claim must respect that.
- **Contrast is validated per theme, not per variant**, and the brand ramps cap what is
  promisable. Never guarantee a ratio a brand cannot deliver — flag it for per-brand validation.
- **Token mapping tables are the designer's job.** Do not demand one in a CRD.
- **Figma cannot express everything.** Where a behaviour is code-only, say so and state what
  Figma approximates it with.
- **WCAG numbers**: 1.4.3 is text contrast, 1.4.11 is non-text, 1.4.4 is Resize Text and not
  contrast. Cite from the table in `<accessibility agent>`.

## Phase 4 — Draft

Write from `crd-template.md` into `<crd path>/{Name}_CRD.md`. Take the section headings from the
template verbatim — do not reorder, rename, merge or invent them, and do not model the file on any
existing CRD.

**Front matter** — you own all of it:

- `id`, `name`, `version: 0.1.0`, `last_updated` today
- `lifecycle: requested`. Moving to `backlog` is a human decision — ask, do not set it
- `status:` — honest flags. A section you did not fill is `not_started`
- `figma:` if a component set exists; `storybook:` when the component ships. **`storybook:` is the
  code link — there is no `code:` key**
- `related_crds` — `dependency` for anatomy children and parents that enforce on this component,
  `precedent` for structurally similar components. Only what an agent must load; loose
  association belongs in Integration prose

**Reciprocal updates, same commit.** Each dependency adds this component to that CRD's "Used in".
Each precedent adds this component to that CRD's precedent list — consistency is symmetric.
**Update the other CRD's `last_updated` only — never bump its `version`.** A back-link is
front-matter maintenance, not a requirements change, and writing one component should not nudge
the version of every component it cites. Report every file you touched.

**Body**: atomic ID'd statements using anatomy nouns; `[open]` on anything provisional;
`[review]` on anything not machine-testable; Ruled out gets every rejection with its why; Open
questions get owner, By and status. Seed Design decisions with any decisions already made —
dated, with the why, citing precedent by name. Only log what someone could plausibly propose the
opposite of; interview churn is not a decision.

Leave later sections as empty headings. Empty with an honest status flag beats invented content.

## Phase 5 — Verify, then hand over

- [ ] Every statement atomic, testable, and using only anatomy nouns
- [ ] No fact appears twice; nothing restates Figma, the code, or the agent rules
- [ ] Accessibility section passes the boundary test — nothing in it would be true of another component
- [ ] Every hedge became `[open]`; every non-testable statement carries `[review]`
- [ ] Every decision and every Ruled out entry has a why
- [ ] Precedent conflicts are honoured or explicitly overruled in Design decisions
- [ ] Every open question has an owner and a By
- [ ] Enforcement statements sit in the right CRD
- [ ] `related_crds` updated in both directions, and you said which files you touched

Present the draft with three lists surfaced, not buried: **the `[open]` markers awaiting design,
the open questions awaiting answers, and any precedent overrules.** Those are what the human has
to action.

Then state the gate plainly: **ready for dev is computed, never stored** — requirements and
design specs complete, `review: complete`, zero `[open]` markers in section 1, every open
question closed or explicitly deferred. Say what currently stands between this CRD and that.

## Shipping

<!-- ADAPT. The mechanics below are one system's. What transfers: CRD changes
     ship like code, through review, and the CRD file is staged explicitly. -->

Every change goes through a branch and a pull request, and every pull request needs a linked work
item in its description — CI fails without one. `<ticket skill>` creates it. Stage the CRD file
explicitly, never `git add -A`.

**Everything else — the branch and commit sequence, the commit-message format, and what to do
when a push is denied — is in `crd-update` § Ship.** Do not restate it here; that section
and this one drifted apart once already.
