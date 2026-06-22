---
name: user-story-standard
description: >-
  Company standard for converting product requirements into standardized user-story files and Jira
  issues. Use when a PM/PO wants to turn requirements (Excel, CSV, PDF, Word, a pasted table, or raw
  text) into individual story .md files with a fixed format, or to create standardized Jira issues
  from those requirements. Triggers include "apply the guidebook", "convert this to user stories",
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
| **Format** | Section order, heading names, frontmatter fields, checkbox syntax, en-dash vs hyphen, slug naming, the Jira naming convention | Follow exactly - no exceptions |
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

## Workflow - analysis first, generate second

Follow these steps in order. **Nothing is written to disk until the analysis is approved (step 5).**

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

### Step 3 - Parse the input
- Excel/CSV: each row is usually one item; map columns to fields.
- PDF/Word: find item boundaries by headings, numbering, or section breaks.
- Pasted text: parse directly.
- Detect each item's `issue_type` (Story / Bug / Spike / Task / Deployment).
- **Edge cases - flag, do not guess:** merged cells, one row holding several items, blank rows
  (skip), image-only PDF with no extractable text (stop and say so), non-English input (flag for
  confirmation), IDs already present in the source (ignore them - we do not use synthetic IDs).

### Step 4 - Analysis pass: write `review.md` (NO story files yet)
Produce `review.md` in the Output folder using `assets/review-template.md`. It must contain:
1. **Proposed items** - a numbered list of `title` / `issue_type` / `epic` for every item.
2. **Ambiguity flags** - anything where persona, goal, benefit, or `issue_type` cannot be determined.
3. **Dependency graph + validation** - list every `Blocked by` / `Enables` relationship by slug, and
   flag any reference that does **not** resolve to another item in this batch (dangling link).
4. **Cross-story consistency** - flag any two items that contradict each other.
5. **Duplicates** - flag near-duplicate items and propose merge/keep.
6. **Risk notes** - items that look oversized, vague, or like a task/spike rather than a story.

Enum fields (`priority`, `category`, `issue_type`) that cannot be determined are flagged HERE and
resolved with the PO - they must hold a real value before generation, never `TBD`.

### Step 5 - PO approves the analysis
Show the `review.md` summary and **wait**. Resolve every 🔴 NEEDS INPUT item with the PO. If the PO
asks to combine, split, rename, or reassign epics, update `review.md` and re-show it. Loop until the
PO approves. Do not generate any story files before approval.

### Step 6 - Generate the story files
Write one `.md` per approved item, applying `references/format-spec.md` exactly:
- Slug filename under the epic folder: `Output/epic-<slug>/<story-slug>.md`.
- All frontmatter fields present (`jira_key` empty); all checkboxes unchecked; three Technical Notes
  subsections present; dependencies as bold labels with slugs.
- **Collision rule:** if the target slug file already exists, show a diff and ask the PO to **skip**,
  **overwrite**, or **write under a new slug**. Never silently clobber a file (it may hold edits).

### Step 7 - Completion summary
List the files written and their locations.

### Step 8 - Post-generation QA
Re-read each file. Silently fix **format only** (field order, dividers, checkbox state, subsection
presence) - **never** silently fill content. Then emit the **Flags Summary** (format in
`references/format-spec.md`). Because content gaps were resolved at step 5, this is normally just
🔵 technical placeholders for the dev team and ⬜ skipped rows.

### Step 9 - Optional Jira push
Only if the PM asks. Read `references/jira-push.md` and follow it. It discovers Jira custom-field IDs
(never hardcoded), prints a dry-run table, requires explicit confirmation before any write, resolves
dependencies into real issue links, and writes the resulting `jira_key` back into each file.

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
