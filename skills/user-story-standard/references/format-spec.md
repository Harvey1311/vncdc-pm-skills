# Format Specification

The binding format for every story file. Read this in full before generating. The format is
non-negotiable; content is your judgment (see SKILL.md).

**Dash rule:** this standard is ASCII-only. Use a plain hyphen `-` everywhere. Do not use the en-dash
(U+2013) or the em-dash (U+2014) - they are hard to type, easy to confuse, and add no value (Jira
renders them identically). Plain hyphens keep output identical across every operator and machine.

**Spelling rule:** the Bug section uses British spelling - `Behaviour` (the field labels **Current
Behaviour** / **Expected Behaviour**), never the American `Behavior`. This is the final, canonical
spelling; keep it identical everywhere the word appears in this skill and in generated output.

---

## 1. Frontmatter

Every story begins with a YAML frontmatter block. Eight fields, always in this exact order:

```yaml
---
title: Story Title in Title Case
epic: Epic Name
issue_type: Story
priority: High
category: Total New
story_points: 5
milestone: M3 - Phase Name
jira_key:
---
```

### Field rules

| Field | Format | Allowed values / constraints |
|-------|--------|------------------------------|
| `title` | Title Case string | Matches the kebab-case slug in the filename |
| `epic` | Epic Name | Plain Epic name; matches the Jira Epic exactly |
| `issue_type` | enum | `Story` \| `Bug` \| `Spike` \| `Task` \| `Deployment` |
| `priority` | enum | `High` \| `Medium` \| `Low` |
| `category` | enum | `Total New` \| `Partial Functional Adjustment` \| `UI Only` |
| `story_points` | integer or `TBD` | Plain integer, no quotes (`3`, `8`, `13`), or `TBD` |
| `milestone` | `M# - Name` or `TBD` | Plain hyphen. Multi-phase: join with ` + ` |
| `jira_key` | Jira key or empty | Empty until pushed; set by the push step (e.g. `KAI-123`) |

### Enum fields never hold `TBD`
`issue_type`, `priority`, and `category` must hold a real enum value. If a value cannot be determined
from the input, flag it in `review.md` and resolve it with the PO **before** generating - never write
an enum as `TBD`. Only `story_points` and `milestone` may be `TBD`.

### Category for a Bug
The `category` enum is feature-shaped, so a defect needs a default: a Bug is normally
`Partial Functional Adjustment` (it corrects existing behaviour). Use `UI Only` when the fix is purely
presentational, and `Total New` only when the defect sits in not-yet-released code. Apply this default
silently - do not flag every Bug's `category` as NEEDS INPUT; flag only when the fix genuinely could be
more than one of these.

### Multi-phase milestone
When a story spans milestones (e.g. a silent backend deploy plus a later UI milestone):
```
milestone: M0 - Silent Backend + M3 - Config Builder (UI)
```

---

## 2. Section order

Every story contains exactly these sections in this order:

```
[YAML frontmatter]

## User Story

---

## Acceptance Criteria

---

## Definition of Done

---

## Dependencies
```

**Horizontal rule (`---`) placement:** after User Story, after Acceptance Criteria, and after
Definition of Done (immediately before Dependencies). **NOT** between the closing `---` of the YAML
block and `## User Story` - the YAML delimiter is not a section divider.

**Bug variant:** when `issue_type` is `Bug`, the first section is `## Bug Details` instead of
`## User Story` (see Section 3). Section order, HR placement, and the remaining three sections are
otherwise identical. Spike / Task / Deployment keep the `## User Story` heading.

---

## 3. User Story

```
**As a** [persona],
**I want to** [action/capability],
**So that** [benefit/outcome].
```

- Three lines, no blank lines between them.
- Comma after lines 1 and 2; period at the end of line 3.
- Use `**As an**` when the persona begins with a vowel sound ("analyst", "admin", "existing user").
- Persona is plain prose (no backticks or bold inside).
- The capability starts with an infinitive action verb ("access", "view", "select", "configure").
- Do NOT write `**I want to** the system to...` - the subject after "I want to" must be an infinitive
  verb, not a noun phrase.

### Non-Story issue types

**Bug** - replace the `## User Story` section with a `## Bug Details` section holding two fields:

```
## Bug Details
**Current Behaviour:** [what happens today - the defect, one line]

**Expected Behaviour:**
- [each expected outcome as a plain bullet]
- [...]
```

- Bold field labels. **Current Behaviour** is a one-line prose statement of the defect (a short
  bulleted list is fine if there are several distinct symptoms). **Expected Behaviour** is a **plain
  bulleted list** - one expected outcome per bullet, never a checkbox (`- [ ]`), consistent with the
  no-checkbox rule for all Jira output.
- Each Expected Behaviour bullet must be concrete and observable - a result the fix can be verified
  against, not `works correctly` or `behaves as expected`. Use **bold** for UI-literal text and
  `backticks` for routes/fields/identifiers, same as AC bullets.
- **Expected Behaviour vs Acceptance Criteria - no duplication.** Expected Behaviour lists the target
  outcomes (what "fixed" looks like); the Acceptance Criteria are the verifiable test conditions that
  prove them (specific inputs, boundaries, endpoints, error strings, regression/edge cases). Keep them
  at different altitudes; do not restate an Expected Behaviour bullet verbatim as an AC, and never let
  the two contradict.
- If either field cannot be determined from the input, write `[BRACKETS]` and flag it - never invent.
- State the substitution (`## User Story` -> `## Bug Details`) in `review.md`.

**Spike / Task / Deployment** - keep the `## User Story` heading but replace the three-line block with
a one-line problem/goal statement. State this substitution in `review.md`.

---

## 4. Acceptance Criteria

### Heading format
```
### AC# - [Criterion Title]
```
- Sequential from AC1 (AC1, AC2, AC3 ...).
- Plain hyphen between the number and the title.
- Title is a short Title Case phrase.

### Checkbox syntax
```
- [ ] Criterion text here
```
Always unchecked. Never pre-check a box. (In the `.md` these stay as Markdown checkboxes; on a Jira
push the Acceptance Criteria and Definition of Done render as plain bullets - the `[ ]` is stripped, see
`jira-push.md`.)

### Writing style
Declarative present tense - state what IS true after implementation.
- Good: `System validates email format on blur/submit`
- Good: `User can select multiple tiers simultaneously`
- Bad: `System should validate` (never "should")
- Bad: `The system will validate` (never future tense)

### Make every criterion concrete and testable
A criterion that cannot be checked is not a criterion. Turn abstract statements into specific,
verifiable ones. If the input does not give the concrete figure, write `[BRACKETS]` and flag it -
do not leave it vague, and do not invent a number.

| Vague (reject) | Concrete (write this) |
|----------------|------------------------|
| System should be fast | Page loads in under `2s` on a 3G connection |
| Users can search | User can find orders by customer name, date range, or status |
| Good error handling | On network failure, retry `3x` then show an offline message |
| It must be secure | Session times out after `15 min` of inactivity |

### Formatting inside AC bullets
- **Bold** for UI-literal text (exact button labels, error messages, screen headings, status values):
  ```
  - [ ] High-contrast **"Start Free Trial"** button is displayed
  - [ ] Error message shown: **"Maximum of 3 active schedules reached."**
  ```
- **Backticks** for routes, endpoints, field names, identifiers, config keys:
  ```
  - [ ] Clicking "Sign In" navigates to `/login`
  - [ ] System calls `GET /api/reports/latest`
  ```
- Numbered sub-lists are allowed inside an AC bullet for an ordered sequence.

---

## 5. Definition of Done

`- [ ]` always unchecked. The DoD has a **universal core** plus **profile items**.

### Universal core (every story, every stack)
```markdown
- [ ] All acceptance criteria implemented and verified
- [ ] No runtime or console errors
- [ ] Code reviewed and merged to `[target-branch]` via MR from `[source-branch]`
```

### Profile items (from the chosen stack profile)
The default VNCDC web profile adds, in order, between the first and last core items:
```markdown
- [ ] TypeScript compiles without errors (`tsc --noEmit`)
- [ ] English & Traditional Chinese (zh) translations added:
  `key.name1`, `key.name2`
```
A non-TS or no-i18n project drops/replaces these per `stack-profiles.md`. `[target-branch]` and
`[source-branch]` are project placeholders to fill before dev handoff.

### Story-specific DoD items (add when applicable)
```markdown
- [ ] DB migration runs cleanly on existing data
- [ ] Unit test for [function name] (see `automation_test/unit-test-instructions.md`)
- [ ] Integration test written for [scenario] (see `automation_test/api-integration-test-instructions.md`)
- [ ] Tested: [scenario] leads to [expected outcome]
```

---

## 6. Dependencies

References use **slugs** (the kebab-case filename without `.md`), not synthetic IDs. They resolve to
real Jira issue links at push time.

```markdown
- **Blocked by:** [slug, slug, ... or none]
- **Enables:** [slug, slug, ... or none]
```

Both lines always present. Empty value is `none` (lowercase), with an optional parenthetical:
```markdown
- **Blocked by:** none (backend is fully independent)
- **Enables:** authentication-pages, landing-page-cta
```

A slug here MUST resolve to another story in the same batch, or it is a dangling reference - flag it
in `review.md`. A parenthetical qualifier per slug is allowed:
```markdown
- **Blocked by:** user-auth-backend (M0), tier-config-backend (M0)
```

---

## 7. Slug and folder naming

### Slugify rule (used for every slug: story files, epic folders, the output folder)
Derive a slug from text **deterministically**, so the same input always yields the same slug on every
machine and operator:

1. Lowercase the text.
2. Replace each **run of non-alphanumeric characters** (spaces, underscores, dots, slashes,
   ampersands, existing hyphens, etc.) with a **single hyphen** `-`.
3. Trim any leading or trailing hyphen.

Examples: `Sprint 17_v1.0` -> `sprint-17-v1-0`; `Report & Export` -> `report-export`;
`Re-generate AI Switch` -> `re-generate-ai-switch`.

Do **not** implement this as "delete the punctuation" - deleting collapses tokens (`v1.0` -> `v10`)
and diverges between operators, which breaks the byte-identical-output and re-run-safety guarantees.
Always convert a run of non-alphanumerics to one hyphen.

"Alphanumeric" means ASCII `a-z` and `0-9`. Translate or transliterate any non-ASCII text (Vietnamese,
CJK, accented letters) to ASCII first - the non-English handling in SKILL.md Step 3 already routes such
input to an English `title` before a slug is derived, so a slug is normally built from ASCII text. If
slugifying would yield an empty string, do not invent one - flag the item for a PO-provided title.

### Story file name
```
<story-slug>.md
```
- `<story-slug>` = the `title` run through the Slugify rule.
- Examples: `landing-page.md`, `config-builder-summary.md`, `regeneration-ai-switch.md`.

### Epic folder name
```
epic-<epic-slug>/
```
- `<epic-slug>` = the epic name run through the Slugify rule.
- Examples: `epic-landing-page/`, `epic-report-configuration-builder/`.

### Output directory structure
The output folder is resolved by SKILL.md (see its **Output layout** section): an existing `Output/`
in the working directory, else the input file's own directory, with a `<source-slug>-user-stories/`
subfolder inside (`<source-slug>` = the input filename without extension, slugified per section 7).
That resolved folder is `<output-dir>` below.
```
<output-dir>/
├── review.md
└── epic-<epic-slug>/
    ├── <story-slug>.md
    └── <story-slug>.md
```
One subfolder per epic when items span multiple epics.

---

## 8. Flags Summary (Step 8 output)

After generation, output a consolidated list of what needs human attention before dev handoff:

```
-- Flags Summary --------------------------------------------
[RED] NEEDS INPUT (cannot be finalised without PO decision)
  <slug> - [Title]: [what's missing or ambiguous]

[YELLOW] NEEDS REVIEW (generated with assumptions - verify before use)
  <slug> - [Title]: [what was assumed]
    -> [field/section]: [the assumption]

[BLUE] TECHNICAL PLACEHOLDERS (dev team must fill in)
  <slug> - [Title]: contains [N] placeholder(s)
    -> [Acceptance Criteria / DoD]: [what needs to be specified]

[SKIPPED] (not converted - explain why)
  Row [N] - "[original text]": [reason skipped]
-------------------------------------------------------------
```

Omit any empty category. If every story is clean: `All [N] stories are ready for dev handoff. No flags.`
With analysis-first, NEEDS INPUT should normally be empty (resolved at step 5); leftover NEEDS INPUT
means generation ran before approval - stop and fix.

**Severity guide:** NEEDS INPUT = incomplete, PO must provide info. NEEDS REVIEW = inferred content,
PO should verify. TECHNICAL PLACEHOLDERS = format fine, dev fills `[BRACKETS]`. SKIPPED = not
converted (heading row, duplicate, pure task, etc.).

---

## Quick rules cheatsheet

| Rule | Value |
|------|-------|
| Dashes | plain hyphen `-` everywhere; no en-dash (U+2013), no em-dash (U+2014) |
| Identity | slug (pre-push) then `jira_key` (post-push); no synthetic `US-###` |
| Checkbox | `- [ ]` always unchecked |
| issue_type values | `Story`, `Bug`, `Spike`, `Task`, `Deployment` |
| Priority values | `High`, `Medium`, `Low` |
| Category values | `Total New`, `Partial Functional Adjustment`, `UI Only` |
| Enum unknown | flag in review.md, resolve before generation - never `TBD` |
| story_points / milestone unknown | `TBD` allowed |
| Empty dependency | `none` (lowercase) |
| Bug first section | `## Bug Details`: **Current Behaviour** (prose) + **Expected Behaviour** (plain bullets); replaces `## User Story` |
| Jira output bullets | Acceptance Criteria + Definition of Done render as plain bullets, no `- [ ]` checkbox, all issue types (stripped on push) |
| Section dividers | `---` after the first section (User Story / Bug Details), AC, DoD - not before it |
