# Worked Example

One complete story conforming to the standard. Content is synthetic; the format is exact. Pattern-
match generated files against this.

**File name:** `user-profile-settings.md`
**Saved to:** `<output-dir>/epic-user-settings/user-profile-settings.md` (e.g. `Output/acme-requirements-user-stories/epic-user-settings/...`; `<output-dir>` is resolved per SKILL.md Output layout)
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
- The DoD follows the canonical assembled order (`format-spec.md` section 5): core first item, then the
  story-specific test item, then the VNCDC Web profile items, then the two remaining core items.
- `Blocked by` references another story by **slug** (`user-authentication`), which must exist in the
  same batch or be flagged as dangling.

---

## Bug example

A Bug uses `## Bug Details` (Current + Expected Behaviour) in place of `## User Story`; everything else
is identical. Note how Expected Behaviour states a concrete, observable result the fix is verified
against - not "works correctly".

**File name:** `expired-otp-accepted-at-login.md`
**Saved to:** `<output-dir>/epic-authentication/expired-otp-accepted-at-login.md`
**Profile:** VNCDC Web (default)

```markdown
---
title: Expired OTP Accepted At Login
epic: Authentication
issue_type: Bug
priority: High
category: Partial Functional Adjustment
story_points: 2
milestone: M2 - Authentication Hardening
jira_key:
---

## Bug Details
**Current Behaviour:** A one-time passcode is still accepted at login after its `5 min` validity window has passed.

**Expected Behaviour:**
- An OTP submitted after its validity window is rejected and login is denied
- The user is prompted to request a new code

---

## Acceptance Criteria

### AC1 - Expiry Enforced
- [ ] An OTP submitted after `5 min` from issue is rejected by `POST /api/auth/otp/verify`
- [ ] A still-valid OTP (within `5 min`) continues to verify successfully

### AC2 - User Feedback
- [ ] On an expired OTP, the login form shows **"Code expired. Request a new one."**
- [ ] The expired code is invalidated server-side and cannot be retried

---

## Definition of Done
- [ ] All acceptance criteria implemented and verified
- [ ] Regression test: expired OTP -> rejected; valid OTP -> accepted
- [ ] TypeScript compiles without errors (`tsc --noEmit`)
- [ ] English & Traditional Chinese (zh) translations added:
  `auth.otpExpired`
- [ ] No runtime or console errors
- [ ] Code reviewed and merged to `develop` via MR from `fix/otp-expiry`

---

## Dependencies
- **Blocked by:** none
- **Enables:** none
```

### What to notice (Bug)
- `## Bug Details` replaces `## User Story`; the other three sections and all format rules are unchanged.
- Expected Behaviour lists the target outcomes as plain bullets (no checkboxes); the checkable
  specifics - the exact error string **"Code expired. Request a new one."**, the
  `POST /api/auth/otp/verify` endpoint, the still-valid path, server-side invalidation - live in the
  ACs. Outcomes are not restated verbatim in both: Expected Behaviour is what "fixed" looks like, the
  ACs are how it is proven.
- `category` for a defect fix is normally `Partial Functional Adjustment`, not `Total New`.
