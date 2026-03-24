# AI Context Layer — Initialization & Usage Guide

---

## Part 1: Global Setup — Cursor + OpenCode (once, on your machine)

Do this before touching any repo.

**1. Create the Cursor global rules folder if it doesn't exist**
```bash
mkdir -p ~/.cursor/rules
```

**2. Paste the two skill files** from the master doc:
- `~/.cursor/rules/update-context.mdc`
- `~/.cursor/rules/plan-feature.mdc`

**3. Set up OpenCode to share the same files via symlinks**
```bash
mkdir -p ~/.opencode/instructions

ln -s ~/.cursor/rules/update-context.mdc ~/.opencode/instructions/update-context.md
ln -s ~/.cursor/rules/plan-feature.mdc    ~/.opencode/instructions/plan-feature.md
```

**4. Create `~/.opencode.json`** with the contents from the master doc.

**5. Verify Cursor picks up global rules**
Open any workspace in Cursor → open the AI chat → type `@` and confirm
`update-context` and `plan-feature` appear as available rules.

Global setup is done. You never touch these files again unless you want to
improve the skill instructions themselves.

---

## Part 1b: Global Setup — Claude Code (once, on your machine)

**1. Create the Claude Code global commands folder**
```bash
mkdir -p ~/.claude/commands
```

**2. Paste the two skill files** from the master doc:
- `~/.claude/commands/update-context.md`
- `~/.claude/commands/plan-feature.md`

**3. Verify Claude Code picks them up**
Open any repo in Claude Code → type `/` and confirm `update-context` and
`plan-feature` appear in the command list.

That's it. Global commands are available in every repo from here on.

> **Tip:** Unlike Cursor, Claude Code doesn't do codebase indexing.
> These skills + the `.context/` folder are your index. Build it once, maintain it.

---

## Part 2: Initializing a New Repo

Run this once per service repo. Works for all three tools (Cursor, OpenCode, Claude Code).

**Step 1 — Create the .context folder**
```bash
cd your-service-repo
mkdir -p .context
```

**Step 2 — Add the three template files** from the master doc:
```bash
touch .context/SERVICE.md
touch .context/PAST_LEARNINGS.md
touch .context/CHANGELOG.context.md   # leave this empty for now
```

**Step 3 — Add tool entry-point files**

For Cursor: add `.cursorrules` (paste the template from the master doc, fill in SERVICE_NAME)

For Claude Code: add `CLAUDE.md` and `AGENTS.md` (paste the templates from the master doc,
fill in SERVICE_NAME in both)

**Step 4 — Optionally add the git hook**
```bash
cp /path/to/post-merge-template .git/hooks/post-merge
chmod +x .git/hooks/post-merge
```

**Step 5 — Do the first context population**

This is the only time you run the update skill on an empty SERVICE.md.
The skill will analyse the repo and populate §1–§7 from actual code.

In Cursor:
```
@update-context
```

In OpenCode:
```bash
opencode "update-context"
```

In Claude Code:
```
/update-context
```

**Step 6 — Review and fill in the human-maintained sections**

After the skill populates the auto-generated sections, open `.context/SERVICE.md`
and manually fill in:
- **§8 Key Architectural Decisions** — the most important section. Write down *why*
  things are the way they are. Think: what would trip up a new engineer or an AI
  that only reads the code?
- **§9 Integration Checklist** — the repeatable pattern specific to this repo.

Commit everything:
```bash
git add .context/ .cursorrules CLAUDE.md AGENTS.md
git commit -m "chore: initialize ai context layer"
```

Repo is initialized. Takes about 10–15 minutes total.

---

## Part 3: Day-to-Day Usage

### Starting a research or planning session

Instead of explaining the service each time, just run:

Cursor / OpenCode:
```
@plan-feature integrate Stripe payments into order-service and downstream
```

Claude Code:
```
/plan-feature integrate Stripe payments into order-service and downstream
```

The skill will:
1. Read SERVICE.md and PAST_LEARNINGS.md automatically
2. Surface any relevant past gotchas as a ⚠️ Watch Out block
3. Do an architecture exploration with no code — data flow, schema impact,
   downstream effects, external API docs if needed

You never explain the service topology again.

### After merging a feature or significant change

Cursor / OpenCode:
```
@update-context
```

Claude Code:
```
/update-context
```

The skill checks git log against the last recorded SHA in CHANGELOG.context.md.
If nothing meaningful changed in `src/` or `infra/`, it exits immediately — no
prompt credit consumed. If there is drift, it rewrites only the affected sections.

### After a non-obvious decision or a gotcha

Manually append an entry to `.context/PAST_LEARNINGS.md` using the entry format.
The update skill will also prompt you to do this when it detects a non-trivial change.
Tags are important — they're how the plan skill matches relevant learnings to future tasks.

Good times to add an entry:
- You hit an unexpected bug during implementation that changed your approach
- You consciously chose not to do something the obvious way
- A third-party API behaved differently than documented
- A change broke a downstream service you didn't anticipate

---

## Part 4: Ongoing Maintenance

| Situation | Cursor / OpenCode | Claude Code |
|---|---|---|
| New feature merged to main | `@update-context` | `/update-context` |
| Starting any new feature or research | `@plan-feature [description]` | `/plan-feature [description]` |
| Made a non-obvious decision | Append entry to `PAST_LEARNINGS.md` | Append entry to `PAST_LEARNINGS.md` |
| Skill instructions feel stale | Edit `~/.cursor/rules/update-context.mdc` | Edit `~/.claude/commands/update-context.md` |
| New engineer joins, new repo | Follow Part 2 — ~15 min setup | Follow Part 2 — ~15 min setup |
| Repo gets significantly restructured | `@update-context --full` + review manually | `/update-context --full` + review manually |

---

## Part 5: Prompt Reference

| What you want | Cursor / OpenCode | Claude Code |
|---|---|---|
| Plan a new integration, no code | `@plan-feature integrate [tool] into [service]` | `/plan-feature integrate [tool] into [service]` |
| Explore architecture options | `@plan-feature explore options for [problem]` | `/plan-feature explore options for [problem]` |
| Update context after a merge | `@update-context` | `/update-context` |
| Research an external API as part of planning | `@plan-feature integrate [tool] — fetch their docs and map to our schema` | `/plan-feature integrate [tool] — fetch their docs and map to our schema` |
| Add a learning after a session | Manually edit `.context/PAST_LEARNINGS.md` | Manually edit `.context/PAST_LEARNINGS.md` |
