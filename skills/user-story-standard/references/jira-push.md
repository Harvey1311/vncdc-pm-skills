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

## Step B - Field discovery and value resolution (never hardcode, never guess)

Custom-field IDs *and* the values that go in native fields differ per project. The `.md` holds **names**
(`epic: User Settings`, `priority: High`); Jira needs **ids/keys/allowed values**. Discover both and
resolve names to real identifiers before building the map.

1. `getJiraProjectIssueTypesMetadata` for the target project - confirm that **every** issue type the
   batch actually uses (Story, Bug, Spike, Deployment, ...) exists, and get their ids. A type the batch
   needs but the project lacks (`Spike` and `Deployment` often do not exist in a default project) is a
   **dry-run blocker**, not a create-time API failure - surface it with options (map to the nearest
   existing type, or the PM adds it).
2. `getJiraIssueTypeMetaWithFields` for each issue type - discover required fields and the real
   custom-field IDs (**Story Points**, **Epic Link**, any other required field). This metadata also
   returns **allowedValues** for Priority, Fix Version, and Issue Type - use it to validate values, not
   just to confirm a field exists (see step 5).
3. If a required field has no source in the `.md`, flag it and ask the PM before proceeding.
4. Resolve the two ambiguous **field** mappings now, before building the map: does this project model
   `milestone` as **Fix Version** or **Sprint**, and does `epic` map to the **Epic Link** custom field
   or to a **parent** issue? Confirm both with the project metadata or the PM.
5. Resolve every native-field **value** from a name to a real id/key/allowed value:
   - **Epic**: search for the Epic by name (`searchJiraIssuesUsingJql`, `issuetype = Epic`) to get its
     issue **key**. No match -> **STOP and ask** the PM to create the Epic or correct the name; do
     **NOT** auto-create an Epic (v1 attaches to pre-existing Epics only). More than one match -> STOP
     and disambiguate. Never guess a key.
   - **Priority, Fix Version, Issue Type**: validate the `.md` value against the allowedValues from
     step 2. An unmatched value (e.g. project has no `Medium`) is a **dry-run blocker** (map to the
     nearest existing value, or the PM adds it) - never write an enum the project does not define.
   - **Sprint** (only if `milestone` maps to Sprint): confirm the sprint exists via the project
     metadata, or ask. The MCP may not expose a sprint list - degrade to PM confirmation rather than
     guessing a sprint id.

## Step C - Build the field map

For each story `.md`, map fields to Jira:

| `.md` source | Jira field |
|--------------|------------|
| `title` | Summary (plain title convention - no prefix) |
| User Story + Acceptance Criteria + Definition of Done | Description |
| `issue_type` | Issue Type |
| `priority` | Priority |
| `story_points` | Story Points (discovered custom field) |
| `epic` | Epic Link / parent - use the Epic **key** resolved in Step B.5 (never the name string; never auto-create the Epic) |
| `milestone` | Fix Version or Sprint - use the version/sprint resolved in Step B (not the raw name) |
| `category` | Label - slugify the enum: lowercase, spaces to hyphens (`Total New` -> `total-new`, `Partial Functional Adjustment` -> `partial-functional-adjustment`, `UI Only` -> `ui-only`) |
| `Dependencies` (`Blocked by` / `Enables`) | Real issue links (Step F) - NOT description text |

Render the Description from the `.md` **body only** - strip the YAML frontmatter (those fields map to
native Jira fields above, not into the Description). Keep the section structure (User Story, Acceptance
Criteria as a checklist, Definition of Done). `TBD` and `[BRACKETS]` carry over as-is
so the dev team sees what is still open. On the first real push, **verify** that the `- [ ]`
acceptance-criteria lines render as a real Jira task list (not literal text) after the MCP's
markdown-to-ADF conversion; if they do not, adjust the render and re-confirm.

**`TBD` / `[BRACKETS]` carry through only in the Description free text.** A `TBD` or placeholder
destined for a **native typed field** - `story_points` (numeric), `milestone` (Version/Sprint), or any
enum - must be **omitted from the create call** and listed as a field gap in the dry-run. Writing the
literal string `TBD` into a numeric or version field is rejected by the API. (`story_points` and
`milestone` are explicitly allowed to be `TBD` upstream, so this case will occur - handle it, do not
assume it away.)

## Step D - Dry-run table (mandatory)

**First, run the idempotency pre-scan** so the dry-run tells the truth about what will actually be
created (the create decision must be made here, not silently at create time):
- For each story, read its `jira_key`. A non-empty `jira_key` means it is already in Jira - mark it
  `skip - already <key>`, no search, no create.
- For each story with an **empty** `jira_key`, search Jira by Summary + Epic
  (`searchJiraIssuesUsingJql`). The match bar is an **exact Summary within the target Epic** - JQL `~`
  is a fuzzy text match, so treat anything short of an exact-summary hit as not confident:
  - **exactly one exact match** -> `recover <key>` (write the key back, do not create) - this closes
    the created-but-not-written-back window.
  - **no match** -> `create`.
  - **more than one match, or only fuzzy/partial matches** -> `ambiguous - needs PM`: neither create
    nor recover until the PM resolves it. Auto-creating here risks a duplicate; auto-recovering risks
    linking to the wrong issue - exactly what the pre-scan exists to prevent.

Then print a table of exactly what will happen, one row per story:

```
status | project | issue_type | summary | epic | points | priority | labels | links
```

`status` is one of `create` / `skip - already <key>` / `recover <key>` / `ambiguous - needs PM`. Also
list, separately: any **dependency links** that will be created (Step F); any **field gaps** (required
Jira fields with no `.md` source, including typed fields omitted because the value is `TBD`); and any
**value blockers** from Step B (missing issue type, unmatched priority, unresolved Epic). Every blocker
must be resolved before the gate - they are not "proceed anyway" warnings.

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
   **`ambiguous - needs PM`**: do nothing and do not link it - it should have been resolved before the
   gate; if it somehow reaches here, skip and report it, never create or recover on a guess.
2. **`create`**: call `createJiraIssue` using the field map, capture the returned key, and
   **immediately write it back** into the story's `.md` frontmatter (`jira_key: <KEY>`) before moving to
   the next story. Process stories one at a time so a mid-batch failure leaves every created issue with
   its key already recorded. If a create succeeds but the write-back fails, STOP and report the orphan
   key - do not continue blind, or the Step D pre-scan on the next run is the only thing standing
   between you and a duplicate.
3. **Resolve dependencies to real links.** Once all created/recovered issues have keys:
   - **Discover the link type** with `getIssueLinkTypes` - do not assume it is named "Blocks" or that
     its labels are exactly "blocks" / "is blocked by". Read the real type name and its inward/outward
     labels (the same discover-never-hardcode rule applied to fields).
   - **Pin the semantics:** `Blocked by: X` means this issue *is blocked by* X (inward); `Enables: Y`
     means this issue *blocks* Y (outward). The two are reciprocal - `a` blocked-by `b` and `b`
     enables `a` are the **same** link.
   - **Build one canonical link set** for the batch, collapsing reciprocal pairs so each link appears
     once. Then, before creating each link, fetch the issue's existing links and skip any match on the
     **(link-type, other-issue) pair regardless of direction** - so "b blocks a" is recognised as the
     existing "a is blocked by b" and not duplicated. Create only the missing links with
     `createIssueLink`.
   - A slug with no issue is a dangling reference - it should have been caught in `review.md`; report it
     and skip the link.

## Step G - Report

Output a summary: each story slug, its new `jira_key` (or "skipped - already exists"), and the links
created. Note any field gaps or dangling references that still need PM attention.

## Safety summary
- No write without the dry-run + explicit confirmation, given in a later message (the second gate).
- The dry-run reflects the truth: the pre-scan decides create / skip / recover / ambiguous, and every
  value blocker and field gap is surfaced, **before** the PM confirms - never silently at create time.
- Resolve names to ids/keys/allowed values, never guess: Epic name -> key (no auto-create), enums
  validated against allowedValues, a `TBD` for a typed field omitted (not written literally).
- Never invent a Jira key, a custom-field id, a link type, or an Epic - discover or ask.
- Never blind-duplicate: a present `jira_key` means already in Jira; an empty key is re-checked against
  Jira (exact Summary within Epic); anything ambiguous goes to the PM, never a guessed create/recover.
- Write each key back immediately after its create; a create-without-write-back is an orphan - stop and
  report it.
- Links are real, direction-aware, and deduped against existing links by (type, other-issue) regardless
  of direction - reciprocal `Blocked by` / `Enables` pairs collapse to one link.
