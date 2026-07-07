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
- #[n] [Title]: [Story: persona / goal / benefit | Bug: current behaviour / expected behaviour |
  Spike/Task/Deployment: problem/goal statement | any: issue_type / enum value that cannot be determined]

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
behaviour.

- #[n] and #[m]: [describe the overlap] -> propose: [merge into one | keep separate because they
  capture genuinely distinct behaviours: ...].

(If none: "All items are mutually exclusive; no merges proposed.")

---

## 8. Risk notes + decomposition proposals

- #[n] [Title]: [oversized / vague / looks like a task or spike rather than a story / etc.]

**Oversized items - proposed slices** (advisory; the PO accepts or rejects at approval). Re-partitions
only behaviour the source already states - no invented scope. The proposed slices appear as the items
in section 1 so the other checks can validate them; the split stays advisory (single-story fallback
offered) until the PO accepts it.
- #[n] [Title] looks oversized. Slicing lens: [by workflow step | by operation create/read/update/delete
  | happy-path-then-edge-cases | by user role | by data variation]. Proposed slices:
  - [proposed-title] (`proposed-slug`) - [the source behaviour this slice covers]
  - [proposed-title] (`proposed-slug`) - [...]
  [If a clean slice would need behaviour the source does not state, say so here instead of inventing it.]

(If none: "No risks flagged; no items need slicing.")

---

## 9. Requirement quality (testability + non-functional)

Each requirement must be concrete enough to become a testable, measurable acceptance criterion, and
any implied non-functional need must be surfaced - not invented.

**Untestable / unmeasurable source** (would force a vague AC or a `[TBD - ...]` placeholder at
generation - needs a concrete figure or condition from the PO):
- #[n] [Title]: [what is too vague - e.g. "fast", "secure", "user-friendly" with no measurable bar]

**Bugs with an unverifiable expected result** (Expected Behaviour too vague to verify a fix against -
needs the concrete observable result from the PO; never invent it):
- #[n] [Title]: [e.g. expected result is "works correctly" with no stated end state]

**Implied non-functional requirements** (performance / security / accessibility the source implies but
does not state - PO to confirm or drop; never invented):
- #[n] [Title]: [the implied NFR - e.g. public page implies rate-limiting; PII implies encryption]

(If none: "Every requirement is concrete enough for a testable AC; no unstated NFRs surfaced.")

---

## 10. E2E validation ticket (advisory)

If the batch forms a coherent multi-story journey under one Epic, propose an optional end-to-end
validation ticket that tests the journey and the seams between the stories (never their unit ACs). Advisory
- the PO accepts or declines; it does not block approval.

- **Proposed:** [yes / no]
- **Journey it covers:** [the set -> apply -> observe flow, in one line]
- **Stories it would stitch (Blocked by):** [slug, slug, ...]
- **Proposed title / slug:** Validate [Feature] End-to-End (`validate-[feature]-end-to-end`)

(If not applicable: "Batch is a single story or unrelated items; no E2E validation ticket proposed.")

---

## Decision needed

Reply with approval, or tell me which items to combine, split, rename, re-type, or reassign to a
different epic. I will update this review and re-show it. Generation starts only after you approve.
