# Format Specification

The binding format for every story file. Read this in full before generating. The format is
non-negotiable; content is your judgment (see SKILL.md).

**Dash rule:** this standard is ASCII-only. Use a plain hyphen `-` everywhere. Do not use the en-dash
(U+2013) or the em-dash (U+2014) - they are hard to type, easy to confuse, and add no value (Jira
renders them identically). Plain hyphens keep output identical across every operator and machine.

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

## Technical Notes

---

## Definition of Done

---

## Dependencies
```

**Horizontal rule (`---`) placement:** after User Story, after Acceptance Criteria, after Technical
Notes, and after Definition of Done (immediately before Dependencies). **NOT** between the closing
`---` of the YAML block and `## User Story` - the YAML delimiter is not a section divider.

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

For non-Story issue types (Bug / Spike / Task / Deployment), the User Story block may be replaced with
a one-line problem/goal statement; keep the section heading. State this substitution in `review.md`.

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
Always unchecked. Never pre-check a box.

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

## 5. Technical Notes

### Mandatory subsections (default VNCDC web profile)
Exactly three, in this order. Never omit one, even when empty.
```markdown
### Frontend
### Backend
### Reuse
```
For non-web stacks, the subsection set comes from the chosen profile in `stack-profiles.md`.

### Multi-milestone variant
When a story has a silent backend milestone (M0) separate from the UI milestone, annotate the
subsection headings:
```markdown
### Backend - M0 (Silent Deploy)
### Frontend - M# (Milestone Name)
### Reuse
```
Use only when `milestone` contains `M0 - Silent Backend`.

### Empty subsection wording
```
### Backend
- No backend changes required.
```
```
### Frontend
- No frontend changes required.
```

### Content style
- **Bold labels** anchor each bullet: `**New component:**`, `**Modify:**`, `**Data:**`, `**Route:**`,
  `**Key change:**`, `**New service:**`, `**New endpoint:**`, `**Verify:**`.
- **Backticks** for all paths, component/function/class names, routes, field names, enum values.
- **Code blocks** (fenced, with language) for SQL DDL, TypeScript/JavaScript, JSON.
- Imperative verbs ("Add", "Modify", "Verify", "Ensure"), not hedges ("Consider", "Maybe", "Could").

### Reuse is never empty
Always list existing hooks, services, components, or utilities to reuse, with paths in backticks.

---

## 6. Definition of Done

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

## 7. Dependencies

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

## 8. Slug and folder naming

### Story file name
```
<story-slug>.md
```
- Slug derived from the title: lowercase, spaces to hyphens, punctuation removed.
- Examples: `landing-page.md`, `config-builder-summary.md`, `regeneration-ai-switch.md`.

### Epic folder name
```
epic-<epic-slug>/
```
- Slug derived from the epic name: lowercase, spaces to hyphens, punctuation removed.
- Examples: `epic-landing-page/`, `epic-report-configuration-builder/`.

### Output directory structure
```
Output/
├── review.md
└── epic-<epic-slug>/
    ├── <story-slug>.md
    └── <story-slug>.md
```
One subfolder per epic when items span multiple epics.

---

## 9. Flags Summary (Step 8 output)

After generation, output a consolidated list of what needs human attention before dev handoff:

```
-- Flags Summary --------------------------------------------
[RED] NEEDS INPUT (cannot be finalised without PO decision)
  <slug> - [Title]: [what's missing or ambiguous]

[YELLOW] NEEDS REVIEW (generated with assumptions - verify before use)
  <slug> - [Title]: [what was assumed]
    -> [field/section]: [the assumption]

[BLUE] TECHNICAL PLACEHOLDERS (dev team must fill in)
  <slug> - [Title]: Technical Notes contain [N] placeholder(s)
    -> [Frontend / Backend]: [what needs to be specified]

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
| Empty backend | `- No backend changes required.` |
| Section dividers | `---` after User Story, AC, Technical Notes, DoD - not before User Story |
