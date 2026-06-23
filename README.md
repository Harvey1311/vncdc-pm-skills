# VNCDC PM Skills

Claude Code skills for VNCDC PMs and POs. Install once; everyone produces the same standardized
output.

> **Source of truth.** This repo is the authoritative home of the VNCDC user-story standard
> (`skills/user-story-standard/`). It supersedes the original `Guidebook.md` (now archived for
> provenance under `Project 9 - TestMD/Rules/Archive/`). Any review mirror elsewhere is a copy, not
> a source - change the standard here.

## Skills

### `user-story-standard`
Converts product requirements (Excel, CSV, PDF, Word, pasted table, or raw text) into individual
user-story `.md` files in a fixed company format, then optionally creates standardized Jira issues
from them. Analysis-first: it writes a `review.md` and waits for your sign-off before generating any
files, so nothing incomplete reaches disk or Jira.

## Install

In Claude Code:

```
/plugin marketplace add Harvey1311/vncdc-pm-skills
/plugin install user-story-standard@vncdc-pm-skills
```

(You can also point at the full URL: `https://github.com/Harvey1311/vncdc-pm-skills`.)

Restart or start a new session. The skill activates automatically when you ask things like:
- "Convert this file to user stories"
- "Apply the guidebook to this input"
- "Generate user stories from <file>"
- "Create Jira stories from these requirements"

## Update

When the standard is revised, pull the latest:

```
/plugin marketplace update vncdc-pm-skills
```

Everyone who installed from the marketplace gets the change - this is the point of distributing it
centrally rather than copying files around.

## How it works (short version)

1. **Collect context** - epic, stack profile, branch model, i18n, target Jira project (if pushing).
2. **Analyze** - parse the input, write `review.md` (proposed items, ambiguity flags, dependency-graph
   validation, duplicates, risks). Nothing is generated yet.
3. **You approve** - resolve flags, rename/split/merge as needed.
4. **Generate** - one `.md` per item, slug-named, in the company format.
5. **Optional Jira push** - dry-run table, explicit confirmation, then create issues + real
   dependency links, writing each Jira key back into the file.

## Conventions in one breath

- Identity is the **slug** (filename), then the **Jira key** once pushed - no synthetic `US-###`.
- Jira **Summary** = plain Title Case phrase; type/epic/labels carry metadata.
- ASCII-only: plain hyphens, no en-dash or em-dash.
- Format is non-negotiable; content is judgment - flag gaps, never fabricate.

Full rules live in `skills/user-story-standard/` (`SKILL.md` + `references/`).

## Requirements

- Claude Code.
- For the optional Jira push: the Atlassian MCP connected to your Jira.
