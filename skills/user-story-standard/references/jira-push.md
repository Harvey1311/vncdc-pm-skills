# Optional Jira Push

Read this only when the PM explicitly asks to push generated stories into Jira (workflow step 9).
This step creates real, outward-facing tickets - treat it with care. **Never create an issue without
explicit confirmation.**

## Prerequisites

- The Atlassian MCP must be connected. Verify with `atlassianUserInfo` or `getVisibleJiraProjects`.
  If it is not connected, **stop** and tell the PM to connect it - do not fake success or guess keys.
- The stories must already exist as approved `.md` files from step 6.
- The target Jira project key must be known (e.g. `KAI`, `MKL`, `ODS`).

## Step A - Preflight

1. Call `getVisibleJiraProjects` (or `atlassianUserInfo`) to confirm access and the cloud id.
2. Confirm the target project key is one the PM can write to.

## Step B - Field discovery (never hardcode field IDs)

Custom-field IDs differ per project. Do not assume them.

1. `getJiraProjectIssueTypesMetadata` for the target project - confirm the issue types you need
   (Story, Bug, etc.) exist and get their ids.
2. `getJiraIssueTypeMetaWithFields` for each issue type - discover required fields and the real
   custom-field IDs for **Story Points** and **Epic Link** (and any other required field).
3. If a required field has no source in the `.md`, flag it and ask the PM before proceeding.

## Step C - Build the field map

For each story `.md`, map fields to Jira:

| `.md` source | Jira field |
|--------------|------------|
| `title` | Summary (plain title convention - no prefix) |
| User Story + Acceptance Criteria + Technical Notes + Definition of Done | Description |
| `issue_type` | Issue Type |
| `priority` | Priority |
| `story_points` | Story Points (discovered custom field) |
| `epic` | Epic Link / parent (match the Jira Epic by name; if absent, ask before creating one) |
| `milestone` | Fix Version or Sprint (confirm which the project uses) |
| `category` | Label (e.g. `total-new`, `ui-only`) |
| `Dependencies` (`Blocked by` / `Enables`) | Real issue links (Step F) - NOT description text |

Render the Description from the `.md` body. Keep the section structure (User Story, Acceptance
Criteria as a checklist, Technical Notes, Definition of Done). `TBD` and `[BRACKETS]` carry over as-is
so the dev team sees what is still open.

## Step D - Dry-run table (mandatory)

Before any write, print a table of exactly what will be created, one row per story:

```
project | issue_type | summary | epic | points | priority | labels | links
```

Also list, separately, any **dependency links** that will be created (Step F) and any **field gaps**
(required Jira fields with no `.md` source).

## Step E - Confirmation gate (hard requirement)

Show the dry-run table and **wait for an explicit "yes"**. Do not call `createJiraIssue` until the PM
confirms. If the PM wants changes, edit the `.md` files (re-run the relevant steps), regenerate the
dry-run, and confirm again.

## Step F - Create issues, then link

1. **Idempotency check first.** For each story, if its `.md` already has a non-empty `jira_key`,
   do NOT create a duplicate. Report it as "already in Jira (`<key>`)" and skip (v1). True update of
   an existing issue is a v2 capability.
2. Create each issue with `createJiraIssue` using the field map. Capture the returned key.
3. **Write the key back** into the story's `.md` frontmatter: `jira_key: <KEY>`.
4. **Resolve dependencies to real links.** Once all issues in the batch have keys, translate each
   `Blocked by` / `Enables` slug to the created issue's key and create the link with
   `createIssueLink` (e.g. "is blocked by" / "blocks"). A slug that has no created issue is a
   dangling reference - it should have been caught in `review.md`; report it and skip the link.

> Workspace hook quirk: run `createIssueLink` and `addCommentToJiraIssue` in **separate turns**,
> never in the same parallel batch - parallel calls trigger false content-integrity denials.

## Step G - Report

Output a summary: each story slug, its new `jira_key` (or "skipped - already exists"), and the links
created. Note any field gaps or dangling references that still need PM attention.

## Safety summary
- No write without the dry-run + explicit confirmation.
- Never invent a Jira key, a custom-field id, or an Epic - discover or ask.
- Never blind-duplicate: a present `jira_key` means the story is already in Jira.
- Links are real Jira issue links, created after all issues exist.
