---
title: Validate [Feature] End-to-End
epic: [same Epic as the sibling stories]
issue_type: Story
priority: High
category: Total New
story_points: [integer or TBD]
milestone: [M# - Phase Name or TBD]
jira_key:
---

## User Story
**As a** business owner of [feature],
**I want** the complete [user] journey validated end-to-end across [roles / tenants / states it spans],
**So that** we have one executable guarantee the feature works as a whole before release.

---

## Acceptance Criteria

*Scope: this ticket verifies cross-story journeys and seams. Per-story behaviour is verified in
[the sibling slugs or Jira keys] and is deliberately not re-listed here.*

### E2E1 - [Journey Title]  (stitches [slug-a] -> [slug-b])
- **Given** [starting state - tenant, role, stored value]
- **When** [the action, or chain of actions across the stories]
- **Then** [the observable business outcome]
- **And** [the seam assertion - the thing only the wired-together stories prove]

### E2E2 - [Journey Title]  (stitches [slug-b] -> [slug-c])
- **Given** [starting state]
- **When** [action chain]
- **Then** [observable outcome]
- **And** [seam assertion]

---

## Definition of Done
- [ ] All E2E scenarios pass on the local stack (`[run command]`)
- [ ] Business-owned prose scenarios recorded in [the project's e2e spec location] (the scenarios above are the source)
- [ ] Executable suite implemented in [the project's e2e test suite], one test per scenario, using the project's e2e framework (e.g. Playwright / k6)
- [ ] Test data seeded: [tenants / accounts / roles the journeys require]
- [ ] No runtime or console errors during runs
- [ ] Reviewed and merged to `[target-branch]` via MR from `[source-branch]`

---

## Dependencies
- **Blocked by:** [every sibling story it validates]
- **Enables:** none

<!--
This is the E2E validation ticket - read references/e2e-validation-ticket.md before filling it in.
Altitude rule: every scenario must cross a SEAM between >=2 sibling stories. If a scenario could be
verified inside one story alone, it belongs in that story, not here. Never restate a sibling's unit AC.
Acceptance Criteria use Given-When-Then as plain bullets (scoped exception, no checkbox); normal stories
stay declarative with checkboxes. Point the DoD at THIS project's own e2e locations and tooling.
-->
