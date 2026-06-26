---
title: [Story Title in Title Case]
epic: [Epic Name]
issue_type: [Story | Bug | Spike | Task | Deployment]
priority: [High | Medium | Low]
category: [Total New | Partial Functional Adjustment | UI Only]
story_points: [integer or TBD]
milestone: [M# - Phase Name or TBD]
jira_key:
---

## User Story
**As a** [persona],
**I want to** [action/capability],
**So that** [benefit/outcome].

---

## Acceptance Criteria

### AC1 - [Criterion Title]
- [ ] [Declarative present-tense statement of what is true after implementation]

### AC2 - [Criterion Title]
- [ ] [Criterion text]

---

## Definition of Done
- [ ] All acceptance criteria implemented and verified
- [ ] TypeScript compiles without errors (`tsc --noEmit`)
- [ ] English & Traditional Chinese (zh) translations added:
  `[key.name1]`, `[key.name2]`
- [ ] No runtime or console errors
- [ ] Code reviewed and merged to `[target-branch]` via MR from `[source-branch]`

---

## Dependencies
- **Blocked by:** [slug or none]
- **Enables:** [slug or none]

<!--
Bug variant: when issue_type is Bug, replace the "## User Story" section above with:

## Bug Details
**Current Behaviour:** [what happens today - the defect, one line]

**Expected Behaviour:**
- [each expected outcome as a plain bullet]

Spike / Task / Deployment: keep "## User Story" but use a one-line problem/goal statement.
-->
