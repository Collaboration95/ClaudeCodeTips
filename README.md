# AI Context Layer

A portable context system for agentic coding tools. Keeps your AI grounded in your service's architecture, past decisions, and gotchas — without re-explaining anything each session.

Works with **Cursor**, **OpenCode**, and **Claude Code**.

---

## Docs

- [`AI-CONTEXT-LAYER.md`](AI-CONTEXT-LAYER.md) — full setup, file-by-file breakdown
- [`repo-init-user-guide.md`](repo-init-user-guide.md) — step-by-step init and day-to-day usage

---

## Sample Structure

Copy these files to the locations shown. Global skill files go in your home directory once. Per-repo files go in each service repo.

```
sample-structure/
│
│   # Global — copy to ~/.claude/commands/
├── .claude/
│   └── commands/
│       ├── update-context.md       ← /update-context  (Claude Code)
│       └── plan-feature.md         ← /plan-feature    (Claude Code)
│
│   # Global — copy to ~/.cursor/rules/
├── .cursor/
│   └── rules/
│       ├── update-context.mdc      ← @update-context  (Cursor)
│       └── plan-feature.mdc        ← @plan-feature    (Cursor)
│
│   # Global — copy to ~/.opencode.json
├── .opencode.json
│
│   # Per-repo — add to every service repo
├── .context/
│   ├── SERVICE.md                  ← architecture, data flow, integrations
│   ├── PAST_LEARNINGS.md           ← append-only gotcha log
│   └── CHANGELOG.context.md        ← SHA tracking for drift detection
│
├── .cursorrules                    ← Cursor entry point
├── CLAUDE.md                       ← Claude Code entry point
└── AGENTS.md                       ← Claude Code skill registry
```
