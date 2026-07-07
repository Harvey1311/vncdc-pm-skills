# E2E Validation Ticket

The binding pattern for an **end-to-end validation ticket**: one ticket that proves a feature works as a
whole, across the stories that build it. Read this in full before proposing or generating one. Loaded on
demand - only when a batch forms a coherent multi-story journey under one Epic, or when the PO asks for an
E2E ticket.

**Dash rule:** ASCII-only, same as the rest of the standard. Plain hyphen `-` everywhere; no en-dash
(U+2013), no em-dash (U+2014).

---

## 1. What it is (and what it is not)

An E2E validation ticket is a **Story sibling** under the same Epic as the feature stories. It is the ticket
that stitches those stories into the real user journey and verifies the **seams between them** - the
hand-offs that no single story can test on its own.

| It IS | It is NOT |
|-------|-----------|
| A journey-altitude check across ≥2 sibling stories | A re-list of the sibling stories' unit acceptance criteria |
| Proof the feature is coherent end-to-end (business + technical) | A place to add new feature scope |
| The last ticket built (blocked by all the stories it validates) | A substitute for each story's own ACs, unit tests, or DoD |

**The altitude rule is the whole point.** Each feature story already verifies its own behaviour in
isolation (its ACs, its unit/integration tests). The E2E ticket earns its place only by testing what those
stories cannot: that when chained into one journey, the hand-offs hold. If a scenario here could be verified
inside a single sibling story, it does not belong here - it belongs in that story.

**Seam test (apply to every scenario):** *"Does this assertion depend on two or more stories being wired
together correctly?"* If yes, it is a seam - keep it. If it verifies one story alone, cut it and point to
that story.

---

## 2. Frontmatter

Same eight fields, same order (see `format-spec.md` section 1). The E2E-specific values:

```yaml
title: Validate [Feature] End-to-End
epic: [same Epic as the sibling stories]
issue_type: Story
priority: High
category: Total New
story_points: TBD
milestone: TBD
jira_key:
```

- `issue_type` is **Story** (it sits as a clean sibling under the Epic; it is not a new type).
- `category` is **Total New** - the E2E suite is a net-new test asset. This is a loose but deliberate fit;
  do not flag it as NEEDS INPUT.

---

## 3. Section format (the delta from a normal story)

Section order, HR placement, and the four headings are unchanged (`format-spec.md` section 2). Only the
*content shape* of two sections differs.

### 3.1 User Story - business-owner reframe

Nobody "wants an E2E test" as a user goal, so the persona is the **business owner** of the feature:

```
## User Story
**As the** business owner of [feature],
**I want** the complete [user] journey validated end-to-end across [the roles / tenants / states it spans],
**So that** we have one executable guarantee the feature works as a whole before release.
```

Three lines, same punctuation rules as a normal User Story. This is content, not a format change.

### 3.2 Acceptance Criteria - Given-When-Then scenarios

**Scoped format exception.** Normal stories use declarative checkbox ACs (`### AC# - Title`, present tense).
The E2E ticket instead uses **Given-When-Then scenarios**, because an end-to-end journey *is* a
precondition -> action -> outcome sequence and reads clearly only in that shape. This exception applies to
the E2E ticket **only**; sibling stories keep the declarative format. `format-spec.md` section 4 records the
exception so the two conventions are not read as a contradiction.

Open the section with a one-line **scope banner** (this is the anti-duplication guard, on the ticket itself):

```
## Acceptance Criteria

> Scope: this ticket verifies cross-story journeys and seams. Per-story behaviour is verified in
> [SEA-20..24 / the sibling slugs] and is deliberately not re-listed here.
```

Then each scenario:

```
### E2E1 - [Journey Title]  (stitches [slug-a] -> [slug-b])
- [ ] **Given** [starting state - tenant, role, stored value]
      **When** [the action, or chain of actions across the stories]
      **Then** [the observable business outcome]
      **And** [the seam assertion - the thing only the wired-together stories prove]
```

Rules for scenarios:
- Number them `E2E1, E2E2, ...` (not `AC1`) so the altitude is unmistakable at a glance.
- Name the sibling stories each scenario stitches, in the heading, by slug or Jira key.
- One checkbox per scenario, keeping the `- [ ]` convention (stripped to a bullet on Jira push, same as ACs).
- **Business outcome in prose; technical seam in `backticks`** (endpoints, stored fields, roles) - carrying
  Minh's business-wise + technical-wise split inside one scenario without a new field.
- Concrete and observable, same testability bar as ACs: bold for UI-literal text
  (**"Number Format"**), backticks for routes/fields. No "works correctly".

### 3.3 Definition of Done - the two execution homes

The DoD carries the business/engineer split Minh defined: business owns the prose scenarios, engineers
implement the executable suite.

```
## Definition of Done
- [ ] All E2E scenarios pass on the local stack (`docker compose up --build`)
- [ ] Business-owned prose scenarios recorded in `e2e/specs/` (the scenarios above are the source)
- [ ] Executable suite implemented in `e2e/tests/` (Playwright / k6), one test per scenario
- [ ] Test data seeded: [tenants / accounts / roles the journeys require]
- [ ] No runtime or console errors during runs
- [ ] Reviewed and merged to `[target-branch]` via MR from `[source-branch]`
```

Point at the repo's central `e2e/` homes; do not fork the scenarios into a second source of truth.

### 3.4 Dependencies - runs last

```
## Dependencies
- **Blocked by:** [every sibling story it validates]
- **Enables:** none
```

Blocked by all of them, because the journey cannot pass until each story it chains is built. This encodes
"the E2E ticket is worked last".

---

## 4. Scenario patterns to reach for

Adapted from the requirements-elicitation scenario patterns; use them to reach coherent journey coverage
**without** drifting down into unit ACs.

- **Happy-path spine** - the primary journey the feature exists for, chained across the stories that deliver
  it (set -> apply -> observe). Always the first scenario.
- **Security / isolation variant** - "actor A's data or setting never leaks to actor B", exercised through
  the full journey, not just one endpoint. This is a seam by nature (it spans the setting story and the
  isolation story), so it belongs here.
- **Default / backward-compatibility path** - the journey a user hits when no one took the configuring
  action, proving the feature degrades to the safe default end-to-end.
- **Edge / scale variant** - boundary states along the journey, only where a boundary emerges from two
  stories interacting (not a single story's boundary AC).

Coverage discipline: a scenario that only re-checks one story's happy path, error string, or boundary is a
duplicate - drop it. Every scenario must cross a seam.

---

## 5. Worked example - SEA-19 Site-wide Number Format

Feature: admins choose a per-tenant number format (comma-style `1,234,567.89` or dot-style `1.234.567,89`)
applied site-wide. Built by five sibling stories: SEA-20 maintain setting, SEA-21 apply site-wide, SEA-22
view read-only, SEA-23 enforce tenant isolation, SEA-24 default for unset tenants.

```markdown
---
title: Validate Site-Wide Number Format End-to-End
epic: Site-wide Number Format
issue_type: Story
priority: High
category: Total New
story_points: TBD
milestone: TBD
jira_key:
---

## User Story
**As the** business owner of the number-format feature,
**I want** the complete admin-to-end-user journey validated end-to-end across roles and tenants,
**So that** we have one executable guarantee the feature works as a whole before release.

---

## Acceptance Criteria

> Scope: this ticket verifies cross-story journeys and seams. Per-story behaviour is verified in
> SEA-20..24 and is deliberately not re-listed here.

### E2E1 - Admin Sets Format, Users See It Everywhere  (stitches SEA-20 -> SEA-21)
- [ ] **Given** tenant VN is on comma-style and I am signed in as Market Admin (`pei5566`)
      **When** I set Number Format to dot-style in System Settings > Advanced and save
      **Then** dashboard KPIs, report figures, list counts, and percentages all render **`1.234.567,89`**
      **And** no screen still shows the previous comma-style format (the saved value drives every display).

### E2E2 - Role Continuity  (stitches SEA-20 -> SEA-22)
- [ ] **Given** the admin has saved dot-style for tenant VN
      **When** a C role (`c1_test`) opens System Settings > Advanced
      **Then** the setting shows dot-style, read-only, with no **Save** control
      **And** a direct API write by that role is rejected (what A set is exactly what C sees and cannot change).

### E2E3 - Isolation Across The Journey  (stitches SEA-21 + SEA-23)
- [ ] **Given** tenant VN is on dot-style and tenant KH is on comma-style
      **When** the admin switches to tenant KH
      **Then** KH renders comma-style and VN's earlier change left KH's display and stored value untouched
      **And** a cross-tenant request (URL, payload, or ID) returns nothing from the other tenant.

### E2E4 - Default Path, No Admin Action  (stitches SEA-24 -> SEA-21)
- [ ] **Given** a tenant that has never set a number format, on a freshly migrated database
      **When** any user views numbers anywhere in the product
      **Then** figures render identically to the current `en-US` comma-style behaviour
      **And** no admin interaction occurred anywhere in the journey.

---

## Definition of Done
- [ ] All E2E scenarios pass on the local stack (`docker compose up --build`)
- [ ] Business-owned prose scenarios recorded in `e2e/specs/` (the scenarios above are the source)
- [ ] Executable suite implemented in `e2e/tests/` (Playwright), one test per scenario
- [ ] Test data seeded: tenants VN + KH, accounts `pei5566` (role A) and `c1_test` (role C)
- [ ] No runtime or console errors during runs
- [ ] Reviewed and merged to `[target-branch]` via MR from `[source-branch]`

---

## Dependencies
- **Blocked by:** maintain-tenant-number-format-setting, apply-tenant-number-format-site-wide,
  view-number-format-setting-read-only, enforce-tenant-isolation-for-number-format-setting,
  default-number-format-for-unset-tenants
- **Enables:** none
```

Notice what the four scenarios do **not** contain: they never restate SEA-21's "comma-style renders
`1,234,567.89`", SEA-22's hidden-Save AC, or SEA-24's migration-backfill AC. Each asserts only the seam
(saved value drives display; A's choice is what C sees; one tenant's change spares another; the default path
needs no action). That is coherence without duplication.

---

## 6. Quick rules cheatsheet

| Rule | Value |
|------|-------|
| issue_type | `Story` (sibling under the Epic) |
| category | `Total New` (net-new test asset; do not flag) |
| User Story persona | the business owner of the feature |
| AC format | Given-When-Then scenarios, numbered `E2E#` - scoped exception to the declarative rule |
| Scope banner | required first line of Acceptance Criteria |
| Every scenario | crosses a seam between ≥2 named sibling stories; never restates a unit AC |
| DoD | prose scenarios in `e2e/specs/`, executable suite in `e2e/tests/`, seeded test data |
| Dependencies | Blocked by every sibling story it validates |
| When to create | one per coherent feature Epic; proposed in review Step 4 or on PO request |
