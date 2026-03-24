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

Checks git log since the last recorded SHA and rewrites only the sections of `SERVICE.md` that have drifted. Exits immediately if nothing meaningful changed in `src/` or `infra/` — no prompt credit consumed.

→ [sample-structure/.cursor/rules/update-context.mdc](sample-structure/.cursor/rules/update-context.mdc)

---

## File 2 of 10 — `~/.cursor/rules/plan-feature.mdc`

Loads `SERVICE.md` and `PAST_LEARNINGS.md`, surfaces any relevant past gotchas as a ⚠️ Watch Out block, then does architecture exploration with no code — until you explicitly ask for it.

→ [sample-structure/.cursor/rules/plan-feature.mdc](sample-structure/.cursor/rules/plan-feature.mdc)

---

## File 3 of 10 — `~/.opencode.json`

Points OpenCode at the symlinked instruction files so Cursor and OpenCode share one source of truth. After creating this file, symlink the rules:

```bash
mkdir -p ~/.opencode/instructions
ln -s ~/.cursor/rules/update-context.mdc ~/.opencode/instructions/update-context.md
ln -s ~/.cursor/rules/plan-feature.mdc    ~/.opencode/instructions/plan-feature.md
```

→ [sample-structure/.opencode.json](sample-structure/.opencode.json)

---

## File 4 of 10 — `~/.claude/commands/update-context.md`

Claude Code equivalent of File 1. Invoked with `/update-context`. Same logic as the Cursor version — no frontmatter needed.

→ [sample-structure/.claude/commands/update-context.md](sample-structure/.claude/commands/update-context.md)

---

## File 5 of 10 — `~/.claude/commands/plan-feature.md`

Claude Code equivalent of File 2. Invoked with `/plan-feature [description]`. Same logic as the Cursor version — no frontmatter needed.

→ [sample-structure/.claude/commands/plan-feature.md](sample-structure/.claude/commands/plan-feature.md)

---

## File 6 of 10 — `.context/SERVICE.md` _(per-repo template)_

Architecture, data flow, integrations, and schema for the service. Sections §1–§7 are auto-maintained by the update skill. Sections §8–§9 are human-maintained and intentionally protected from overwrites.

→ [sample-structure/.context/SERVICE.md](sample-structure/.context/SERVICE.md)

---

## File 7 of 10 — `.context/PAST_LEARNINGS.md` _(per-repo template)_

Append-only log of gotchas and non-obvious decisions. Tagged entries are scanned and surfaced by the plan skill before any new feature work starts.

→ [sample-structure/.context/PAST_LEARNINGS.md](sample-structure/.context/PAST_LEARNINGS.md)

---

## File 8 of 10 — `.cursorrules` _(per-repo, Cursor)_

Three lines. Points Cursor at `.context/` and lists the available skills.

→ [sample-structure/.cursorrules](sample-structure/.cursorrules)

---

## File 9 of 10 — `CLAUDE.md` _(per-repo, Claude Code entry point)_

Three lines. Claude Code reads this automatically on session start — just points to `AGENTS.md`.

→ [sample-structure/CLAUDE.md](sample-structure/CLAUDE.md)

---

## File 10 of 10 — `AGENTS.md` _(per-repo, Claude Code skill registry)_

Points Claude Code at the `.context/` files and lists the available `/skills`.

→ [sample-structure/AGENTS.md](sample-structure/AGENTS.md)

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
