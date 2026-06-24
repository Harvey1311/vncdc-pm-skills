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
| **Content** | The user-story sentence, acceptance criteria, Technical Notes detail, DoD specifics | Infer from input, adapt to context, flag gaps - never fabricate |

This is not a stamp. A story with 2 honest acceptance criteria beats one padded to 5. Technical Notes
that say `[BRACKETS]` where the tech is unknown beat invented file paths. When in doubt: **write what
you know, flag what you don't, ask when you're blocked.**

## Before you start - read the standard

This SKILL.md is the orchestrator. The binding rules live in `references/` and you MUST read the
relevant ones in full before generating anything:

- `references/format-spec.md` - frontmatter, section order, story template, acceptance criteria,
  Technical Notes, Definition of Done, dependencies, and slug naming. **Read this in full first.**
- `references/jira-naming-convention.md` - the universal Jira Summary convention.
- `references/worked-example.md` - one complete, format-exact example to pattern-match against.
- `references/stack-profiles.md` - read only when the project's stack is not the default VNCDC web
  stack, or when the PM asks about the DoD / Technical Notes shape.
- `references/jira-push.md` - read only when the PM asks to push to Jira (workflow step 9).
- `assets/blank-template.md` - the empty story to copy.
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
| Stack profile | Decides DoD items + Technical Notes shape (default: VNCDC web - see `stack-profiles.md`) |
| Branch model | DoD merge line (`[target-branch]` / `[source-branch]` placeholders if unknown) |
| i18n requirement | DoD translation item wording (or "no i18n" if not applicable) |
| Target Jira project key | Only if a push is wanted later (e.g. `KAI`, `MKL`, `ODS`) |

If the PO says "use defaults", apply the default VNCDC web profile and `[bracket]` placeholders, and
flag them in the review.

**If the chosen profile is not VNCDC Web (default), read `references/stack-profiles.md` in full now,
before proceeding to Step 3.** Step 6 generates the DoD and Technical Notes subsections against that
profile - it must be loaded before then, not assumed.

### Step 3 - Parse the input
- Excel/CSV: each row is usually one item; map columns to fields.
- PDF/Word: find item boundaries by headings, numbering, or section breaks.
- Pasted text: parse directly.
- Detect each item's `issue_type` (Story / Bug / Spike / Task / Deployment).
- **Record each item's source location** (row number, sheet, section, page) so every story can be
  traced back in the review. Note any source item you do NOT turn into a story, and why.
- **Edge cases - flag, do not guess:** merged cells, one row holding several items, blank rows
  (skip), image-only PDF with no extractable text (stop and say so), non-English input (flag for
  confirmation), IDs already present in the source (ignore them - we do not use synthetic IDs).

### Step 4 - Analysis pass: write `review.md` (NO story files yet)
Produce `review.md` in the Output folder using `assets/review-template.md`. It must contain:
1. **Proposed items** - a numbered list of `title` / `issue_type` / `epic` / slug for every item.
2. **Source traceability** - a map from each story slug to the exact source location it came from,
   plus a list of source items deliberately skipped (with reason). Proves nothing was invented.
3. **Coverage check** - confirm every source requirement maps to at least one story; flag any
   unmapped source item as a potential omission.
4. **Ambiguity flags** - anything where persona, goal, benefit, or `issue_type` cannot be determined.
5. **Dependency graph + validation** - list every `Blocked by` / `Enables` relationship by slug, and
   flag any reference that does **not** resolve to another item in this batch (dangling link).
6. **Cross-story consistency** - flag any two items that contradict each other.
7. **Duplicates / overlaps** - apply the merge test (can two stories be merged without losing
   meaning? if yes, merge); propose merges.
8. **Risk notes** - items that look oversized, vague, or like a task/spike rather than a story.
9. **Requirement quality (testability + non-functional)** - for each item, check the source is concrete
   enough to yield a testable, measurable acceptance criterion. Flag any requirement too vague to
   become a checkable AC (it would otherwise force a vague AC or a `[BRACKETS]` placeholder at
   generation). Also surface any non-functional dimension the source *implies but does not state* -
   performance, security, accessibility - and flag it for the PO to confirm or drop. **Never invent an
   NFR or a concrete measure the source did not provide** - flag it, do not fill it.

Enum fields (`priority`, `category`, `issue_type`) that cannot be determined are flagged HERE and
resolved with the PO - they must hold a real value before generation, never `TBD`.

### Step 5 - PO approves the analysis
Show the `review.md` summary, then **STOP and end your turn** per the Approval protocol above - do not
continue to Step 6 in the same message, and do not treat a clean (zero-flag) review as approval.

Resolve every 🔴 NEEDS INPUT item with the PO. Batch independent questions in one shot; ask
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
- Slug filename under the epic folder: `Output/epic-<slug>/<story-slug>.md`.
- All frontmatter fields present (`jira_key` empty); all checkboxes unchecked; three Technical Notes
  subsections present; dependencies as bold labels with slugs.
- **Collision rule:** if the target slug file already exists, show a diff and ask the PO to **skip**,
  **overwrite**, or **write under a new slug**. Never silently clobber a file (it may hold edits).
- **`jira_key` hard stop:** if the existing file carries a **non-empty `jira_key`**, this is not a plain
  collision - the story is already live in Jira and `jira_key` cannot be regenerated (Jira assigns it;
  you never invent it). Do **not** offer a silent overwrite. Surface the `jira_key`, state that
  overwriting destroys the Jira linkage, and require explicit PO direction before doing anything.

### Step 7 - Completion summary
List the files written and their locations.

### Step 8 - Post-generation QA
Re-read each file. Silently fix **format only** (field order, dividers, checkbox state, subsection
presence) - **never** silently fill content. Then emit the **Flags Summary** (format in
`references/format-spec.md`). Because content gaps were resolved at step 5, this is normally just
🔵 technical placeholders for the dev team and ⬜ skipped rows.

**Tripwire:** if any 🔴 NEEDS INPUT survives to this point, generation ran before the analysis was
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

```
Output/
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
