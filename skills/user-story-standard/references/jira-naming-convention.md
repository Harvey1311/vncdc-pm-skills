# Jira Naming Convention (Universal)

The VNCDC standard for how a Jira issue's **Summary** is written, across every project and issue
type. The Summary is the only human-readable name the skill controls - Jira assigns the issue *key*
(`KAI-123`) automatically.

## The rule: plain title, native fields carry metadata

The Summary is a clean Title Case action phrase describing the outcome. Nothing else. Categorization
lives in Jira-native fields - Issue Type, Epic Link, Labels, Priority - not in the Summary text.

```
Summary:  Resend OTP on Check-in Page
          Lock Account After Five Failed Attempts
          Fix Expired OTP Being Accepted

NOT:      US-030 - Resend OTP ...        (no synthetic ID prefix)
          [Story] Resend OTP ...         (no type prefix - the Type field shows it)
          Check-in: Resend OTP ...       (no area prefix - the Epic shows it)
```

### Why no prefixes
- The Issue Type field already shows Story / Bug / Spike (icon + label) - a `[Story]` prefix is
  redundant noise.
- The Epic Link already groups the issue - an area prefix duplicates it.
- Synthetic IDs (`US-030`) drift, collide on re-runs, and compete with Jira's own key. We dropped
  them on purpose; the slug (pre-push) and the Jira key (post-push) are the only identifiers.

## Writing the Summary

- **Title Case**, outcome-first, imperative or descriptive.
- **No trailing punctuation.**
- **Concise** - aim for under ~80 characters; it must be scannable in a backlog list.
- **Self-explanatory out of context** - a reader scanning the backlog should understand it without
  opening the issue.
- For a **Bug**, describe the fix or the defect plainly: `Fix Expired OTP Being Accepted` or
  `OTP Accepts Expired Code`.
- For a **Spike**, lead with the question: `Evaluate OTP Delivery Vendors`.

## How it maps from the `.md`

The `.md` `title` field IS the Summary. The filename slug is the kebab-case of the same title. So the
title is the single source of truth for both the file identity and the Jira Summary - keep them in
sync. If a title changes, the slug (and any dependency referencing it) changes too.

## Metadata goes to fields, not the Summary

| Information | Where it lives in Jira |
|-------------|------------------------|
| Issue type (Story/Bug/Spike/...) | Issue Type field (from `.md` `issue_type`) |
| Epic grouping | Epic Link (from `.md` `epic`) |
| Priority | Priority field (from `.md` `priority`) |
| Estimate | Story Points field (from `.md` `story_points`) |
| Release/sprint | Fix Version or Sprint (from `.md` `milestone`) |
| Classification (Total New / UI Only / ...) | Label (from `.md` `category`) |

See `jira-push.md` for the exact field discovery and write mechanics.
