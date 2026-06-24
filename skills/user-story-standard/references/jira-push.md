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
4. Resolve the two ambiguous mappings now, before building the map in Step C - do not leave them to
   guess at map time: does this project model `milestone` as **Fix Version** or **Sprint**, and does
   `epic` map to the **Epic Link** custom field or to a **parent** issue? Confirm both with the project
   metadata or the PM.

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
| `category` | Label - slugify the enum: lowercase, spaces to hyphens (`Total New` -> `total-new`, `Partial Functional Adjustment` -> `partial-functional-adjustment`, `UI Only` -> `ui-only`) |
| `Dependencies` (`Blocked by` / `Enables`) | Real issue links (Step F) - NOT description text |

Render the Description from the `.md` **body only** - strip the YAML frontmatter (those fields map to
native Jira fields above, not into the Description). Keep the section structure (User Story, Acceptance
Criteria as a checklist, Technical Notes, Definition of Done). `TBD` and `[BRACKETS]` carry over as-is
so the dev team sees what is still open. On the first real push, **verify** that the `- [ ]`
acceptance-criteria lines render as a real Jira task list (not literal text) after the MCP's
markdown-to-ADF conversion; if they do not, adjust the render and re-confirm.

## Step D - Dry-run table (mandatory)

**First, run the idempotency pre-scan** so the dry-run tells the truth about what will actually be
created (the create decision must be made here, not silently at create time):
- For each story, read its `jira_key`. A non-empty `jira_key` means it is already in Jira - mark it
  `skip - already <key>`, it will not be re-created.
- For each story with an **empty** `jira_key`, guard the create/write-back failure window: search Jira
  by Summary + Epic (`searchJiraIssuesUsingJql`). A confident match means an issue was created on a
  previous run but the key never got written back - mark it `recover <key>` (write the key back, do not
  create a duplicate). No match means `create`.

Then print a table of exactly what will happen, one row per story:

```
status | project | issue_type | summary | epic | points | priority | labels | links
```

`status` is one of `create` / `skip - already <key>` / `recover <key>`. Also list, separately, any
**dependency links** that will be created (Step F) and any **field gaps** (required Jira fields with no
`.md` source).

## Step E - Confirmation gate (hard requirement)

This is the **second gate** and obeys the Approval protocol in `SKILL.md`. Show the dry-run table, then
**STOP and end your turn.** Write nothing - call no `createJiraIssue` / `createIssueLink` - until the
PM confirms explicitly in a **later message**. Never infer confirmation from silence, from a clean
dry-run with no field gaps, or from the original push request. **Even if the dry-run is clean, you still
stop.** End with an explicit ask, e.g. "Reply YES to create these in Jira, or tell me what to change."
If the PM wants changes, edit the `.md` files (re-run the relevant steps), regenerate the dry-run, and
confirm again.

## Step F - Create issues, then link

Act on the Step D status for each story; do not re-decide here.

1. **`skip - already <key>`**: do nothing, the issue exists. (True update of an existing issue is a v2
   capability.) **`recover <key>`**: write the matched key back into the `.md`, do NOT create.
2. **`create`**: call `createJiraIssue` using the field map, capture the returned key, and
   **immediately write it back** into the story's `.md` frontmatter (`jira_key: <KEY>`) before moving to
   the next story. Process stories one at a time so a mid-batch failure leaves every created issue with
   its key already recorded. If a create succeeds but the write-back fails, STOP and report the orphan
   key - do not continue blind, or the Step D pre-scan on the next run is the only thing standing
   between you and a duplicate.
3. **Resolve dependencies to real links.** Once all issues in the batch have keys, translate each
   `Blocked by` / `Enables` slug to its issue key. **Before creating any link, fetch the issue's
   existing links and skip any that already exist** - re-running a batch must not duplicate links (issue
   creation is guarded by status, link creation needs its own guard). Create only the missing links with
   `createIssueLink` (e.g. "is blocked by" / "blocks"). A slug with no issue is a dangling reference - it
   should have been caught in `review.md`; report it and skip the link.

> Workspace hook quirk: run `createIssueLink` and `addCommentToJiraIssue` in **separate turns**,
> never in the same parallel batch - parallel calls trigger false content-integrity denials.

## Step G - Report

Output a summary: each story slug, its new `jira_key` (or "skipped - already exists"), and the links
created. Note any field gaps or dangling references that still need PM attention.

## Safety summary
- No write without the dry-run + explicit confirmation, given in a later message (the second gate).
- The dry-run reflects the truth: the idempotency pre-scan decides create / skip / recover before the
  PM confirms, never silently at create time.
- Never invent a Jira key, a custom-field id, or an Epic - discover or ask.
- Never blind-duplicate: a present `jira_key` means already in Jira; an empty key is re-checked against
  Jira (Summary + Epic) to catch a created-but-not-written-back issue before creating.
- Write each key back immediately after its create; a create-without-write-back is an orphan - stop and
  report it.
- Links are real Jira issue links, created after all issues exist, and only if they do not already
  exist.
