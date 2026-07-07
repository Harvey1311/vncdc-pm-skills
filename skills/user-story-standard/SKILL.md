---
name: user-story-standard
description: >-
  Company standard for converting product requirements into standardized user-story files and Jira
  issues. Use when a PM/PO wants to turn requirements (Excel, CSV, PDF, Word, a pasted table, or raw
  text) into individual story .md files with a fixed format, or to create standardized Jira issues
  from those requirements. Triggers include "apply the user-story standard", "convert this to user stories",
  "generate user stories from <file>", "process this input", "standardize these stories", "create
  Jira stories/issues from <requirements>". Do NOT use for editing an existing single Jira ticket's
  wording, for writing PRDs or specs, or for non-story documents.
license: Proprietary - VNCDC internal use
---

# User Story Standard

The VNCDC company standard for turning product requirements into individual user-story `.md` files
and, optionally, standardized Jira issues. Every file and issue produced shares the same structure,
style, and naming, regardless of project, stack, or operator.

## Format vs Content - the one rule that governs everything

**The format is non-negotiable. The content requires judgment.**

| Layer | What it covers | Your role |
|-------|----------------|-----------|
| **Format** | Section order, heading names, frontmatter fields, checkbox syntax, hyphen usage (ASCII-only), slug naming, the Jira naming convention | Follow exactly - no exceptions |
| **Content** | The user-story sentence (or a Bug's current/expected behaviour), acceptance criteria, DoD specifics | Infer from input, adapt to context, flag gaps - never fabricate |

This is not a stamp. A story with 2 honest acceptance criteria beats one padded to 5. An acceptance
criterion that says `[TBD - ...]` where a concrete figure is unknown beats an invented number. When in
doubt: **write what you know, flag what you don't, ask when you're blocked.**

## Before you start - read the standard

This SKILL.md is the orchestrator. The binding rules live in `references/` and you MUST read the
relevant ones in full before generating anything:

- `references/format-spec.md` - frontmatter, section order, story template, acceptance criteria,
  Definition of Done, dependencies, and slug naming. **Read this in full first.**
- `references/jira-naming-convention.md` - the universal Jira Summary convention.
- `references/worked-example.md` - one complete, format-exact example to pattern-match against.
- `references/stack-profiles.md` - read only when the project's stack is not the default VNCDC web
  stack, or when the PM asks about the DoD shape.
- `references/jira-push.md` - read only when the PM asks to push to Jira (workflow step 9).
- `references/e2e-validation-ticket.md` - read only when a batch forms a coherent multi-story journey
  under one Epic (proposed in Step 4), or when the PM asks for an end-to-end validation ticket.
- `assets/blank-template.md` - the empty story to copy.
- `assets/e2e-ticket-template.md` - the empty E2E validation ticket to copy.
- `assets/review-template.md` - the shape of the pre-generation `review.md`.

## Identity model - no synthetic numbers

Stories are NOT numbered (`US-001` etc. are gone). Identity is:

- **Before push:** the **slug** (the kebab-case filename, e.g. `resend-otp-on-check-in-page`).
- **After push:** the **Jira key** (e.g. `KAI-123`), written back into the file's `jira_key`
  frontmatter field. Jira assigns the key; you never invent it.

Dependencies reference other stories by **slug** until push, then resolve to real Jira issue links.
This is what makes re-runs safe: a repeated slug is a real "same story twice" signal, and a present
`jira_key` means "already in Jira - do not duplicate."

## Approval protocol - the rule that governs every gate

This skill has two human gates: the **analysis approval** (step 5) and the **Jira push confirmation**
(the dry-run in step 9). Both obey the same rule:

- **Approval is explicit and arrives in a *later* message.** Never infer it from silence, from a clean
  review with no flags, or from the original request. "Convert these and push them" authorizes
  **neither** gate - it is a request, not an approval.
- **A blanket pre-authorization is not approval.** "Just generate them, I trust you, don't wait" given
  *before* the review exists does not authorize generation - you cannot approve an analysis that does
  not exist yet. Approval is only meaningful after the review is presented. Acknowledge the
  instruction, produce the review, then still stop for approval.
- **After presenting anything that needs approval, STOP and end your turn.** Do not continue to the
  next step in the same message. End with an explicit ask - e.g. "Reply APPROVE, or tell me what to
  change" - not a trailing sentence the next step then runs past.
- **Even when the review has zero flags, you still stop.** A clean review is exactly when the model is
  most tempted to roll straight through - that temptation is the failure, not a license.
- The two gates are independent: passing one never passes the other. Two separate explicit approvals,
  always.

## Workflow - analysis first, generate second

Follow these steps in order. **No story files are written to disk until the analysis is approved
(step 5); only `review.md` is written before then.**

### Step 1 - Load the standard
Read `references/format-spec.md`, `references/jira-naming-convention.md`, and
`references/worked-example.md` in full. Do not skip ahead.

### Step 2 - Collect project context
Ask the PO for anything not already provided:

| Item | Why it's needed |
|------|-----------------|
| Epic name(s) | Frontmatter `epic` and folder grouping; matches the Jira Epic name |
| Stack profile | Decides DoD items (default: VNCDC web - see `stack-profiles.md`) |
| Branch model | DoD merge line (`[target-branch]` / `[source-branch]` placeholders if unknown) |
| i18n requirement | DoD translation item wording (or "no i18n" if not applicable) |
| Target Jira project key | Only if a push is wanted later (e.g. `KAI`, `MKL`, `ODS`) |

If the PO says "use defaults", apply the default VNCDC web profile and `[bracket]` placeholders, and
flag them in the review.

**If the chosen profile is not VNCDC Web (default), read `references/stack-profiles.md` in full now,
before proceeding to Step 3.** Step 6 generates the DoD against that profile - it must be loaded
before then, not assumed.

**Resolve the output location now** (smart default - do not ask): pick the destination per the
**Output layout** section below, then state the resolved path in plain text before you write anything.
The skill never assumes a fixed `Output/` exists; it resolves a folder and announces it, and the PO
can redirect. The same resolved folder is reused for `review.md` (Step 4) and the story files (Step 6).

### Step 3 - Parse the input
- Excel/CSV: each row is usually one item; map columns to fields.
- PDF/Word: find item boundaries by headings, numbering, or section breaks.
- Pasted text: parse directly.
- Detect each item's `issue_type` (Story / Bug / Spike / Task / Deployment).
- **Record each item's source location** (row number, sheet, section, page) so every story can be
  traced back in the review. Note any source item you do NOT turn into a story, and why.
- **Edge cases - flag, do not guess:** merged cells, one row holding several items (flag; if it is one
  oversized requirement rather than separate items, carry it as a single item and propose the split in
  Step 4 item 8 - never guess it apart here), blank rows (skip), image-only PDF with no extractable text
  (stop and say so), non-English input (flag for confirmation), IDs already present in the source
  (ignore them - we do not use synthetic IDs).

### Step 4 - Analysis pass: write `review.md` (NO story files yet)
Produce `review.md` in the resolved output folder (see **Output layout**) using
`assets/review-template.md`. It must contain:
1. **Proposed items** - a numbered list of `title` / `issue_type` / `epic` / slug for every item.
2. **Source traceability** - a map from each story slug to the exact source location it came from,
   plus a list of source items deliberately skipped (with reason). Proves nothing was invented.
3. **Coverage check** - confirm every source requirement maps to at least one story; flag any
   unmapped source item as a potential omission.
4. **Ambiguity flags** - anything where the fields a given `issue_type` needs cannot be determined:
   for a Story, persona / goal / benefit; for a **Bug**, current behaviour or expected behaviour; for
   a Spike / Task / Deployment, the one-line problem/goal statement; and for any type, the `issue_type`
   itself. Flag, do not guess.
5. **Dependency graph + validation** - list every `Blocked by` / `Enables` relationship by slug, and
   flag any reference that does **not** resolve to another item in this batch (dangling link).
6. **Cross-story consistency** - flag any two items that contradict each other.
7. **Duplicates / overlaps** - apply the merge test (can two stories be merged without losing
   meaning? if yes, merge); propose merges.
8. **Risk notes + decomposition proposals** - flag items that look oversized, vague, or like a
   task/spike rather than a story. When an item is **oversized** - its source describes several
   distinct user-facing behaviours, spans multiple workflow steps, or would need more acceptance
   criteria than one story can coherently carry - do not stop at the flag: **propose** how to slice it
   into smaller, independently shippable stories. This is advisory: you propose, the PO disposes.
   - Name the slicing lens you used (by workflow step, by operation create/read/update/delete, by
     happy-path-then-edge-cases, by user role, or by data variation) and list the candidate slices as
     proposed titles + slugs, each a thin vertical slice that could ship on its own.
   - **Re-partition only behaviour the source already states. Never propose a slice for scope the
     source is silent on** - flag the gap instead; inventing a slice is fabrication, same rule as the
     rest of this skill.
   - List the proposed slices in section 1 as the items, so the coverage, dependency, and consistency
     checks (sections 2-7) validate the intended set - but state plainly that the split is advisory:
     name the single-story fallback ("I will generate one story unless you accept the split") and
     recommend, do not impose. Generate no files until the PO accepts at Step 5; if they accept, the
     split is confirmed there and the full analysis re-runs on the agreed set (per the Approval
     protocol). A proposed slice is a suggestion, not a `[RED]` gate - it never blocks approval on its own.
9. **Requirement quality (testability + non-functional)** - for each item, check the source is concrete
   enough to yield a testable, measurable acceptance criterion. Flag any requirement too vague to
   become a checkable AC (it would otherwise force a vague AC or a `[TBD - ...]` placeholder at
   generation). Also surface any non-functional dimension the source *implies but does not state* -
   performance, security, accessibility - and flag it for the PO to confirm or drop. **Never invent an
   NFR or a concrete measure the source did not provide** - flag it, do not fill it.
   For a **Bug**, apply the same testability bar to each **Expected Behaviour** bullet: each must state
   a concrete, observable result the fix can be verified against (e.g. *expired OTP is rejected with*
   **"Code expired"**), not a vague *works correctly* / *behaves as expected*. Flag a Bug whose expected
   outcomes are too vague to verify, and **never invent an expected result** the source did not give.
10. **E2E validation ticket proposal (advisory)** - if the batch forms a coherent multi-story journey
    under **one Epic** (the stories set -> apply -> observe some capability, or several stories must be
    wired together for the feature to work), **propose** an optional end-to-end validation ticket that
    tests the journey and the seams between those stories. This is advisory - you propose, the PO
    disposes; it never blocks approval. Read `references/e2e-validation-ticket.md` before proposing or
    generating one. Also honour a direct PO request for an E2E ticket at any time. Do **not** propose one
    for a batch of unrelated stories or a single story. If proposed and the PO accepts, generate it in
    Step 6 as a Story sibling under the Epic (`assets/e2e-ticket-template.md`), blocked by the stories it
    validates - and re-run the dependency check so its `Blocked by` list resolves.

Enum fields (`priority`, `category`, `issue_type`) that cannot be determined are flagged HERE and
resolved with the PO - they must hold a real value before generation, never `TBD`.

### Step 5 - PO approves the analysis
Show the `review.md` summary, then **STOP and end your turn** per the Approval protocol above - do not
continue to Step 6 in the same message, and do not treat a clean (zero-flag) review as approval.

Resolve every [RED] NEEDS INPUT item with the PO. Batch independent questions in one shot; ask
sequentially only when one answer changes the next (e.g. confirm `issue_type` before asking for a
persona).

If the PO asks to combine, split, rename, or reassign epics, update `review.md`. **Any rename, split,
or merge changes slugs** (the slug derives from the title, and dependencies reference other stories by
slug). Before re-showing the updated review, **re-run the full Step 4 analysis** on the changed set: a
rename can dangle a `Blocked by:` that validated clean a moment ago, and a split can create a new story
that others should now depend on. Do not re-show without re-validating. Loop until the PO approves. Do
not generate any story files before approval.

### Step 6 - Generate the story files
Write one `.md` per approved item, applying `references/format-spec.md` exactly:
- Slug filename under the epic folder: `<output-dir>/epic-<slug>/<story-slug>.md`, where
  `<output-dir>` is the folder resolved in Step 2 (see **Output layout**).
- All frontmatter fields present (`jira_key` empty); all checkboxes unchecked; dependencies as bold
  labels with slugs.
- **Collision rule:** if the target slug file already exists, show a diff and ask the PO to **skip**,
  **overwrite**, or **write under a new slug**. Never silently clobber a file (it may hold edits).
- **`jira_key` hard stop:** if the existing file carries a **non-empty `jira_key`**, this is not a plain
  collision - the story is already live in Jira and `jira_key` cannot be regenerated (Jira assigns it;
  you never invent it). Do **not** offer a silent overwrite. Surface the `jira_key`, state that
  overwriting destroys the Jira linkage, and require explicit PO direction before doing anything.

### Step 7 - Completion summary
List the files written and their locations, including the resolved output folder path (the same one
announced before generation).

### Step 8 - Post-generation QA
Re-read each file. Silently fix **format only** (field order, dividers, checkbox state) - **never**
silently fill content. Then emit the **Flags Summary** (format in
`references/format-spec.md`). Because content gaps were resolved at step 5, this is normally just
[BLUE] technical placeholders for the dev team and [SKIPPED] rows.

**Tripwire:** if any [RED] NEEDS INPUT survives to this point, generation ran before the analysis was
approved. STOP - do not emit a completion summary - and return to Step 5 for approval.

### Step 9 - Optional Jira push
Only if the PM asks. Read `references/jira-push.md` and follow it. It discovers Jira custom-field IDs
(never hardcoded), prints a dry-run table, requires explicit confirmation before any write, resolves
dependencies into real issue links, and writes the resulting `jira_key` back into each file.

The dry-run confirmation is the **second gate** and obeys the Approval protocol above: present the
dry-run table, STOP and end your turn, and write nothing until the PO explicitly confirms in a later
message. A push request made back at the start does not pre-authorize this - the step 5 approval and
this confirmation are two separate gates.

## Output layout

### Resolving the output folder (before the first write; announce it)
The skill does not assume a fixed `Output/` exists. Resolve the destination once, at the start
(Step 2), and reuse it for every write (`review.md` at Step 4, story files at Step 6):

1. **Base directory** - the first that applies:
   - An `Output/` folder exists in the **current working directory** -> base = that `Output/`.
     ("Current working directory" is wherever the session is running; in a VNCDC project that is the
     project root, so this preserves the existing `Output/` workflow.)
   - Otherwise -> base = the directory that contains the input file.
2. **Output folder** - inside the base, always a folder named from the source file:
   `<base>/<source-slug>-user-stories/`, where `<source-slug>` is the input filename **without its
   extension, run through the Slugify rule** in `format-spec.md` section 7. Example:
   `Q3 Feature Requests.csv` -> `q3-feature-requests-user-stories/`. A deterministic, source-derived name means the same input
   always resolves to the same folder, so the Step 6 collision rule and the `jira_key` hard stop keep
   working across separate sessions. For pasted text with no input file, use `<base>/user-stories/`
   and say so.
3. **Announce, do not ask.** Before the first write, state the **actual resolved path from steps 1-2**
   (the real one - not the placeholder, and not always `Output/...`), e.g. "Writing analysis to
   `<resolved-path>/review.md` - tell me if you want it elsewhere." If the PO names a different path,
   use it verbatim. Repeat the final path in the Step 7 summary.

**Never create a per-session or timestamped folder.** A fresh folder per run orphans earlier output
and silently defeats re-run safety (the collision and `jira_key` guards can only fire if a re-run
lands in the same folder). The folder name comes from the source, never from the session.

### Structure
`<output-dir>` below = `<base>/<source-slug>-user-stories/`, resolved above.

```
<output-dir>/
├── review.md                       (the approved analysis)
└── epic-<slug>/
    ├── <story-slug>.md
    └── <story-slug>.md
```

## A note on dashes
This standard is ASCII-only: use a plain hyphen `-` everywhere (prose, slugs, acceptance-criteria
headings, milestone values). Do not use the en-dash (U+2013) or the em-dash (U+2014) - they are hard
to type, easy to confuse with each other, and add no value (Jira renders them identically). Plain
hyphens keep every operator's output byte-identical, which is the whole point of a shared standard.
