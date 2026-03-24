# AI Context Layer — Master Setup Document
> For Cursor + OpenCode CLI + Claude Code · Polyrepo · Global skills, per-repo context

---

## Architecture Overview

```
## CURSOR + OPENCODE (global + per-repo)

~/.cursor/rules/
├── update-context.mdc     ← @update-context skill
└── plan-feature.mdc       ← @plan-feature skill

~/.opencode/instructions/  ← symlinked to ~/.cursor/rules/
├── update-context.md      ← symlink
└── plan-feature.md        ← symlink

~/.opencode.json           ← global OpenCode config


## CLAUDE CODE (global + per-repo)

~/.claude/commands/
├── update-context.md      ← /update-context skill
└── plan-feature.md        ← /plan-feature skill


## PER-REPO (every service repo, works across all three tools)

your-service-repo/
├── .context/
│   ├── SERVICE.md            ← architecture, data flow, integrations, schema
│   ├── PAST_LEARNINGS.md     ← append-only gotcha/decision log
│   └── CHANGELOG.context.md  ← SHA tracking for drift detection
├── .cursorrules              ← 3 lines pointing Cursor to .context/
├── CLAUDE.md                 ← 3 lines pointing Claude Code to AGENTS.md
└── AGENTS.md                 ← points to .context/, lists available /skills
```

---

## File 1 of 10 — `~/.cursor/rules/update-context.mdc`

```
---
description: Check for code drift and update .context/SERVICE.md if meaningful source changes exist since last update
globs:
alwaysApply: false
---

# @update-context

You are a context document maintainer. Your job is to decide whether
.context/SERVICE.md needs updating, and if so, regenerate only the
sections that have drifted — not the whole document.

---

## Step 1 — Check if an update is needed

Read .context/CHANGELOG.context.md to get the commit SHA of the last context update.

Run:
  git log --oneline [LAST_SHA]..HEAD -- src/ infra/ cmd/ lib/

If output is empty → respond with:
  "No meaningful source changes since last context update ([DATE]). Skipping."
Stop here. Do not proceed.

If there are commits, summarise what changed in 2–3 lines and continue.

---

## Step 2 — Identify which sections need updating

Only regenerate sections affected by the changed files:

| Changed area                        | Sections to update                                          |
|-------------------------------------|-------------------------------------------------------------|
| src/integrations/**                 | §4 Integrations table, §5 Env vars (if new), §6 Schema     |
| src/handlers/** or entry point      | §2 Trigger & Entry Point, §3 Data Flow                     |
| src/s3/** or output types           | §6 Output Schema                                           |
| .env.example or config files        | §5 Environment Variables                                   |
| infra/**                            | §3 Data Flow diagram, §4 Integrations                     |
| Downstream notification code        | §7 Downstream Consumers                                    |

Never regenerate §8 (Architectural Decisions) or §9 (Integration Checklist).
Those are human-maintained.

---

## Step 3 — Regenerate affected sections only

For each affected section:
1. Read the relevant source files
2. Rewrite only that section — match existing format exactly
3. Leave all other sections verbatim

---

## Step 4 — Update CHANGELOG.context.md

Append:
  ## [DATE] — [short reason, e.g. "Added Honeybadger integration"]
  - Sections updated: [list]
  - Commits: [SHA list or titles]
  - New HEAD SHA: [current HEAD]

---

## Step 5 — Output summary

Report:
- What changed in the codebase
- Which sections were updated
- Any inconsistencies found (e.g. new env var in code but missing from .env.example)
- Whether a PAST_LEARNINGS entry should be added — prompt the user if a
  non-obvious decision was made
```

---

## File 2 of 10 — `~/.cursor/rules/plan-feature.mdc`

```
---
description: Load service context and plan a new feature with past-learnings awareness before any implementation
globs:
alwaysApply: false
---

# @plan-feature [description]

You are a senior engineer helping plan a feature or integration for this service.
Before any planning, load context and surface anything that could affect the approach.

---

## Step 1 — Load context

Read in full:
1. .context/SERVICE.md
2. .context/PAST_LEARNINGS.md

Confirm with:
  "Loaded context for [SERVICE_NAME]. Last updated [DATE]."

---

## Step 2 — Scan PAST_LEARNINGS for relevance

Match the tags in PAST_LEARNINGS entries against the feature description:

| Feature type                  | Tags to match                    |
|-------------------------------|----------------------------------|
| New integration               | [integration] [auth] [schema]    |
| Queue / async work            | [sqs] [performance]              |
| Auth / token handling         | [auth]                           |
| Output or data shape changes  | [schema] [s3]                    |
| Infrastructure changes        | [infra]                          |

If relevant entries found, present a ⚠️ Watch Out block first, before any planning:

  ⚠️ RELEVANT PAST LEARNINGS

  1. [DATE] — [Title]
     → Why relevant: [reason tied to this feature]
     → Key takeaway: [one line]

If nothing relevant: state "No past learnings flagged for this feature."

---

## Step 3 — Architecture exploration (no code)

### Overview
What is the feature? What problem does it solve?

### Where it fits in the existing flow
Map it to the current data flow from SERVICE.md. Where does it slot in?

### Architecture additions needed
- New components or files (following the pattern in §9 of SERVICE.md)
- New env vars required
- Schema changes to §6 if applicable
- Downstream consumer impact per §7

### Data available
If this involves a new external tool or API, use web fetch to look up their docs:
- What data/events they expose
- Auth method
- Rate limits or constraints worth knowing

### Open questions / risks
Decisions that need to be made before implementation starts.

---

## Guardrails

- Do NOT write implementation code unless explicitly asked after this planning phase
- Do NOT suggest changing SQS queue names, S3 paths, or completion notification
  format without flagging downstream dependencies (§7 of SERVICE.md)
- Always check §8 (Architectural Decisions) before suggesting structural changes
- If your plan conflicts with an existing architectural decision, flag it explicitly
```

---

## File 3 of 10 — `~/.opencode.json`

```json
{
  "instructions": [
    "~/.opencode/instructions/update-context.md",
    "~/.opencode/instructions/plan-feature.md"
  ]
}
```

> After creating this file, symlink the instruction files so Cursor and OpenCode
> share one source of truth:
>
> ```bash
> mkdir -p ~/.opencode/instructions
> ln -s ~/.cursor/rules/update-context.mdc ~/.opencode/instructions/update-context.md
> ln -s ~/.cursor/rules/plan-feature.mdc    ~/.opencode/instructions/plan-feature.md
> ```

---

## File 4 of 10 — `~/.claude/commands/update-context.md`

> Claude Code global skill. Invoked with `/update-context`.
> Same logic as the Cursor version — no frontmatter needed.

```markdown
# /update-context

You are a context document maintainer. Your job is to decide whether
.context/SERVICE.md needs updating, and if so, regenerate only the
sections that have drifted — not the whole document.

---

## Step 1 — Check if an update is needed

Read .context/CHANGELOG.context.md to get the commit SHA of the last context update.

Run:
  git log --oneline [LAST_SHA]..HEAD -- src/ infra/ cmd/ lib/

If output is empty → respond with:
  "No meaningful source changes since last context update ([DATE]). Skipping."
Stop here. Do not proceed.

If there are commits, summarise what changed in 2–3 lines and continue.

---

## Step 2 — Identify which sections need updating

Only regenerate sections affected by the changed files:

| Changed area                        | Sections to update                                          |
|-------------------------------------|-------------------------------------------------------------|
| src/integrations/**                 | §4 Integrations table, §5 Env vars (if new), §6 Schema     |
| src/handlers/** or entry point      | §2 Trigger & Entry Point, §3 Data Flow                     |
| src/s3/** or output types           | §6 Output Schema                                           |
| .env.example or config files        | §5 Environment Variables                                   |
| infra/**                            | §3 Data Flow diagram, §4 Integrations                     |
| Downstream notification code        | §7 Downstream Consumers                                    |

Never regenerate §8 (Architectural Decisions) or §9 (Integration Checklist).
Those are human-maintained.

---

## Step 3 — Regenerate affected sections only

For each affected section:
1. Read the relevant source files
2. Rewrite only that section — match existing format exactly
3. Leave all other sections verbatim

---

## Step 4 — Update CHANGELOG.context.md

Append:
  ## [DATE] — [short reason, e.g. "Added Honeybadger integration"]
  - Sections updated: [list]
  - Commits: [SHA list or titles]
  - New HEAD SHA: [current HEAD]

---

## Step 5 — Output summary

Report:
- What changed in the codebase
- Which sections were updated
- Any inconsistencies found (e.g. new env var in code but missing from .env.example)
- Whether a PAST_LEARNINGS entry should be added — prompt the user if a
  non-obvious decision was made
```

---

## File 5 of 10 — `~/.claude/commands/plan-feature.md`

> Claude Code global skill. Invoked with `/plan-feature [description]`.
> Same logic as the Cursor version — no frontmatter needed.

```markdown
# /plan-feature [description]

You are a senior engineer helping plan a feature or integration for this service.
Before any planning, load context and surface anything that could affect the approach.

---

## Step 1 — Load context

Read in full:
1. .context/SERVICE.md
2. .context/PAST_LEARNINGS.md

Confirm with:
  "Loaded context for [SERVICE_NAME]. Last updated [DATE]."

---

## Step 2 — Scan PAST_LEARNINGS for relevance

Match the tags in PAST_LEARNINGS entries against the feature description:

| Feature type                  | Tags to match                    |
|-------------------------------|----------------------------------|
| New integration               | [integration] [auth] [schema]    |
| Queue / async work            | [sqs] [performance]              |
| Auth / token handling         | [auth]                           |
| Output or data shape changes  | [schema] [s3]                    |
| Infrastructure changes        | [infra]                          |

If relevant entries found, present a ⚠️ Watch Out block first, before any planning:

  ⚠️ RELEVANT PAST LEARNINGS

  1. [DATE] — [Title]
     → Why relevant: [reason tied to this feature]
     → Key takeaway: [one line]

If nothing relevant: state "No past learnings flagged for this feature."

---

## Step 3 — Architecture exploration (no code)

### Overview
What is the feature? What problem does it solve?

### Where it fits in the existing flow
Map it to the current data flow from SERVICE.md. Where does it slot in?

### Architecture additions needed
- New components or files (following the pattern in §9 of SERVICE.md)
- New env vars required
- Schema changes to §6 if applicable
- Downstream consumer impact per §7

### Data available
If this involves a new external tool or API, use web fetch to look up their docs:
- What data/events they expose
- Auth method
- Rate limits or constraints worth knowing

### Open questions / risks
Decisions that need to be made before implementation starts.

---

## Guardrails

- Do NOT write implementation code unless explicitly asked after this planning phase
- Do NOT suggest changing SQS queue names, S3 paths, or completion notification
  format without flagging downstream dependencies (§7 of SERVICE.md)
- Always check §8 (Architectural Decisions) before suggesting structural changes
- If your plan conflicts with an existing architectural decision, flag it explicitly
```

---

## File 6 of 10 — `.context/SERVICE.md` _(per-repo template)_

```markdown
# Service Context: [SERVICE_NAME]

> **Last updated:** [DATE]
> **Last update reason:** [e.g. "New integration: Datadog added"]
> Auto-generated sections: §1–§7. Do not edit manually.
> Human-maintained sections: §8 Architectural Decisions, §9 Integration Checklist.

---

## §1 One-liner
What this service does in one sentence.

---

## §2 Trigger & Entry Point
- **Trigger type:** SQS / HTTP / Cron / Event
- **Triggered by:** [upstream service or event source]
- **Entry handler:** src/handlers/...

---

## §3 Data Flow

1. Receives [X] from [source]
2. Authenticates via [method]
3. Fetches [Y] from [service/API]
4. Transforms / processes
5. Writes to [S3 / DB / queue]
6. Sends completion notification to [SQS / SNS]

ASCII Diagram:
  [SQS: incident-queue]
          ↓
    [this-service]
     ├─ queries [DataSource A]
     ├─ queries [DataSource B]
     └─ writes → [S3: bucket/{incident_id}/]
          ↓
    [SQS: completion-queue]
     ├─ [consumer-service-A reads]
     └─ [consumer-service-B reads]

---

## §4 Integrations & External Dependencies

| System | Direction | Auth Method | Notes |
|--------|-----------|-------------|-------|
|        |           |             |       |

---

## §5 Environment Variables

  ENV_VAR_ONE=
  ENV_VAR_TWO=

---

## §6 Output Schema (S3 / Event payload)

  {
    "incident_id": "string",
    "source": "...",
    "fetched_at": "ISO8601",
    "raw_data": {},
    "metadata": {}
  }

---

## §7 Downstream Consumers

| Service | Reads From | Expects |
|---------|------------|---------|
|         |            |         |

---

## §8 Key Architectural Decisions
<!-- HUMAN-MAINTAINED — why things are the way they are.
     Prevents AI from suggesting changes that conflict with intentional design. -->

- **[Decision title]:** [Reason]

---

## §9 Adding a New Integration (Checklist)
<!-- HUMAN-MAINTAINED — the repeatable pattern for this specific repo -->

- [ ] Add handler in src/integrations/[name]/
- [ ] Register in src/registry.ts
- [ ] Add env vars to .env.example and §5 above
- [ ] Update §6 if output schema changes
- [ ] Update §7 if new downstream consumers are affected
- [ ] Add a PAST_LEARNINGS entry after completion
```

---

## File 7 of 10 — `.context/PAST_LEARNINGS.md` _(per-repo template)_

```markdown
# Past Learnings & Decision Log

> Append-only. Never delete entries.
> The @plan-feature / /plan-feature skill scans this and surfaces relevant entries before planning.

---

## Entry Format

  ### [YYYY-MM-DD] — [Short Title]
  **Tags:** [integration] [sqs] [auth] [schema] [performance] [infra]
  **Context:** What were we trying to do?
  **What happened / what we learned:**
  **Decision made:**
  **Files affected:**

---

## Entries

<!-- ADD NEW ENTRIES ABOVE THIS LINE -->
```

---

## File 8 of 10 — `.cursorrules` _(per-repo, Cursor)_

```
# [SERVICE_NAME]

Service context is in .context/. Read SERVICE.md and PAST_LEARNINGS.md
before answering any architecture or implementation question.

Skills: @update-context, @plan-feature [description]
```

---

## File 9 of 10 — `CLAUDE.md` _(per-repo, Claude Code entry point)_

```markdown
# [SERVICE_NAME]

See AGENTS.md for context files and available skills.
```

> 3 lines. Claude Code reads this automatically on session start.
> It just needs to point to AGENTS.md — keep it minimal.

---

## File 10 of 10 — `AGENTS.md` _(per-repo, Claude Code skill registry)_

```markdown
# [SERVICE_NAME] — Agent Instructions

## Context
Read before any architecture or implementation question:
- `.context/SERVICE.md` — architecture, data flow, integrations, schema
- `.context/PAST_LEARNINGS.md` — append-only gotcha and decision log

## Available Skills
- `/update-context` — check git log for drift, rewrite only affected sections of SERVICE.md
- `/plan-feature [description]` — load context and plan a feature before any code is written
```

---

## Bonus — `.git/hooks/post-merge` _(optional, per-repo)_

A zero-cost nudge after pulling merged changes. No LLM call — just a reminder.

```bash
#!/bin/bash
CHANGED=$(git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD)

if echo "$CHANGED" | grep -qE '^(src|infra|cmd|lib)/'; then
  echo ""
  echo "⚡ Source changes detected. Consider running @update-context (Cursor) or /update-context (Claude Code)."
  echo ""
fi
```

```bash
# Enable it:
chmod +x .git/hooks/post-merge
```
