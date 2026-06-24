# Worked Example

One complete story conforming to the standard. Content is synthetic; the format is exact. Pattern-
match generated files against this.

**File name:** `user-profile-settings.md`
**Saved to:** `Output/epic-user-settings/user-profile-settings.md`
**Profile:** VNCDC Web (default)

```markdown
---
title: User Profile Settings
epic: User Settings
issue_type: Story
priority: Medium
category: Partial Functional Adjustment
story_points: 3
milestone: M6 - User Experience
jira_key:
---

## User Story
**As a** registered user,
**I want to** update my display name and notification preferences from a settings page,
**So that** I can personalise my experience and control how the platform communicates with me.

---

## Acceptance Criteria

### AC1 - Display Name Update
- [ ] Settings page includes a text input pre-populated with the current display name
- [ ] Saving a new display name updates it across all pages immediately (no page reload required)
- [ ] Display name must be 2-50 characters; inline error shown if outside range: **"Display name must be between 2 and 50 characters"**

### AC2 - Notification Preferences
- [ ] User can toggle email notifications **On** / **Off** for the following events:
  - Report generated
  - Report export ready
- [ ] Preference saved via `PUT /api/users/me/preferences`
- [ ] Toggle state persists on page reload

### AC3 - Save Confirmation
- [ ] On successful save, a toast notification displays: **"Settings saved successfully"**
- [ ] On API failure, toast displays: **"Failed to save settings. Please try again."**

---

## Definition of Done
- [ ] All acceptance criteria implemented and verified
- [ ] Display name validation tested: <2 chars -> error, >50 chars -> error, valid -> saves
- [ ] TypeScript compiles without errors (`tsc --noEmit`)
- [ ] English & Traditional Chinese (zh) translations added:
  `settings.displayName`, `settings.notifications`, `settings.reportGenerated`, `settings.exportReady`, `settings.saveSuccess`, `settings.saveError`
- [ ] No runtime or console errors
- [ ] Code reviewed and merged to `feature/phase3` via MR from `feature/user-settings`

---

## Dependencies
- **Blocked by:** user-authentication (user must be logged in to access settings)
- **Enables:** none
```

## What to notice
- No synthetic `id` field; identity is the slug `user-profile-settings` (and `jira_key` once pushed).
- `issue_type: Story` and empty `jira_key` are present.
- All dashes are plain hyphens.
- The DoD = universal core + VNCDC Web profile items + one story-specific test item.
- `Blocked by` references another story by **slug** (`user-authentication`), which must exist in the
  same batch or be flagged as dangling.
