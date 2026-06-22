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

## 2. Ambiguity flags (must resolve before generation)

[RED] NEEDS INPUT
- #[n] [Title]: [persona / goal / benefit / issue_type / enum value that cannot be determined]

(If none: "No blocking ambiguities.")

---

## 3. Dependency graph + validation

Relationships declared across the batch (by slug):

- [story-slug-a] **blocked by** [story-slug-b]
- [story-slug-c] **enables** [story-slug-d]

**Dangling references** (slug not found in this batch):
- #[n] [Title]: `Blocked by: [unknown-slug]` does not match any item - resolve or remove.

(If none: "All dependency references resolve within the batch.")

---

## 4. Cross-story consistency

- [Any two items that contradict each other, with the conflict described.]

(If none: "No contradictions found.")

---

## 5. Duplicates / near-duplicates

- #[n] and #[m] appear to cover the same feature - propose: [merge | keep separate because ...].

(If none: "No duplicates found.")

---

## 6. Risk notes

- #[n] [Title]: [oversized / vague / looks like a task or spike rather than a story / etc.]

(If none: "No risks flagged.")

---

## Decision needed

Reply with approval, or tell me which items to combine, split, rename, re-type, or reassign to a
different epic. I will update this review and re-show it. Generation starts only after you approve.
