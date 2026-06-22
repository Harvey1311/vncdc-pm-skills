# Conversion Review - [Source file or input name]

Pre-generation analysis. **No story files are written until the PO approves this.**

- **Source:** [input filename / description]
- **Stack profile:** [VNCDC Web (default) | Backend/API | Data/Pipeline | Generic]
- **Target Jira project:** [KEY, or "not pushing"]
- **Date:** [YYYY-MM-DD]

---

## 1. Proposed items

| # | Title | issue_type | epic | slug |
|---|-------|-----------|------|------|
| 1 | [Title] | Story | [Epic] | [story-slug] |
| 2 | [Title] | Bug | [Epic] | [story-slug] |

---

## 2. Source traceability

Every proposed story must trace to a real location in the source. This proves nothing was dropped
or invented, and makes skipped rows visible.

| Story slug | Source location | Notes |
|------------|-----------------|-------|
| [story-slug] | [e.g. Row 4 / Sheet "Stories" / PDF section 2.1] | [verbatim if helpful] |

**Skipped source items** (not turned into a story, with reason):
- [Row N / "original text"]: [heading row | duplicate | not a story | out of scope | etc.]

---

## 3. Coverage check (collective exhaustiveness)

Every requirement in the source must map to at least one proposed story. Flag any source item with
no story.

- **Unmapped source items** (potential omissions): [Row/section that produced no story - confirm it
  is intentional, or a story is missing.]

(If none: "Every source item maps to a story or is listed as a deliberate skip above.")

---

## 4. Ambiguity flags (must resolve before generation)

[RED] NEEDS INPUT
- #[n] [Title]: [persona / goal / benefit / issue_type / enum value that cannot be determined]

(If none: "No blocking ambiguities.")

---

## 5. Dependency graph + validation

Relationships declared across the batch (by slug):

- [story-slug-a] **blocked by** [story-slug-b]
- [story-slug-c] **enables** [story-slug-d]

**Dangling references** (slug not found in this batch):
- #[n] [Title]: `Blocked by: [unknown-slug]` does not match any item - resolve or remove.

(If none: "All dependency references resolve within the batch.")

---

## 6. Cross-story consistency

- [Any two items that contradict each other, with the conflict described.]

(If none: "No contradictions found.")

---

## 7. Duplicates / overlaps (mutual exclusivity)

Apply the merge test to every pair that looks related: **can these two stories be merged without
losing meaning?** If yes, they are not distinct - merge them. Each story must capture one distinct
behavior.

- #[n] and #[m]: [describe the overlap] -> propose: [merge into one | keep separate because they
  capture genuinely distinct behaviors: ...].

(If none: "All items are mutually exclusive; no merges proposed.")

---

## 8. Risk notes

- #[n] [Title]: [oversized / vague / looks like a task or spike rather than a story / etc.]

(If none: "No risks flagged.")

---

## Decision needed

Reply with approval, or tell me which items to combine, split, rename, re-type, or reassign to a
different epic. I will update this review and re-show it. Generation starts only after you approve.
