# Stack Profiles

The format in `format-spec.md` is universal. A few pieces depend on the project's technology stack -
the Technical Notes subsection set and some Definition of Done items. Those live here as **profiles**
so a non-web or non-TypeScript team can adopt the same format without inheriting another team's stack.

Pick the profile in step 2 of the workflow. Default is **VNCDC Web** when the PM says "use defaults."

---

## Universal core (applies to every profile)

These DoD items are stack-independent and are always present, in this order:

```markdown
- [ ] All acceptance criteria implemented and verified
- [ ] No runtime or console errors
- [ ] Code reviewed and merged to `[target-branch]` via MR from `[source-branch]`
```

Profile items are inserted between the first and last core items.

---

## Profile: VNCDC Web (default)

The stack the original standard assumed: TypeScript/React frontend, Java backend, i18n in English +
Traditional Chinese.

**Technical Notes subsections:** `Frontend`, `Backend`, `Reuse`.

**DoD profile items:**
```markdown
- [ ] TypeScript compiles without errors (`tsc --noEmit`)
- [ ] English & Traditional Chinese (zh) translations added:
  `key.name1`, `key.name2`
```

**i18n exception** - when a story has no user-visible text (silent background work), replace the
translation item with:
```markdown
- [ ] No i18n keys required ([brief reason])
```

---

## Profile: Backend / API only

No UI; no i18n.

**Technical Notes subsections:** `Backend`, `Reuse` (omit `Frontend`).

**DoD profile items:**
```markdown
- [ ] [compile/build check for the language, e.g. `mvn compile` or `go build ./...`]
- [ ] No i18n keys required (no user-visible text)
```

---

## Profile: Data / Pipeline

ETL, batch jobs, schema work.

**Technical Notes subsections:** `Pipeline`, `Data`, `Reuse`.

**DoD profile items:**
```markdown
- [ ] Job runs cleanly end-to-end on a representative dataset
- [ ] Schema migration runs cleanly on existing data
- [ ] No i18n keys required (no user-visible text)
```

---

## Profile: Generic / Other

When none of the above fits. Keep the universal core only and ask the PM for the project's
build/verify command and whether i18n applies.

**Technical Notes subsections:** `Implementation`, `Reuse`.

**DoD profile items:** ask the PM; at minimum a build/verify item.

---

## How to use a profile
1. In step 2, set the profile (default VNCDC Web).
2. Apply that profile's Technical Notes subsection set wherever `format-spec.md` says "from the
   chosen profile."
3. Build the DoD as: universal core (first item) + profile items + universal core (remaining items),
   plus any story-specific DoD items from `format-spec.md` section 6.
4. Record the chosen profile in `review.md` so the PO can see and override it.

## Adding a new profile
Profiles are intentionally simple: a Technical Notes subsection list + a short DoD item list. To add
one for a new stack, copy a section above and adjust. Keep the universal core untouched.
