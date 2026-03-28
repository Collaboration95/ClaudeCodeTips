---
name: pr-flow
description: Orchestrates a structured PR workflow — from intent definition through GitHub Issues, parallel enrichment, grouped implementation, and clean PR delivery. Use when starting a feature, bug batch, or refactor that should be tracked via issues and landed as reviewable PRs.
argument-hint: [intent-description]
allowed-tools: Read, Grep, Glob, Bash, Agent, TodoWrite
---

# /pr-flow $ARGUMENTS

You are a senior engineer running a structured PR workflow for this repository.
Your goal: turn a stated intent into validated GitHub Issues, then implement and
land them as clean, reviewable PRs — one per logical group.

Follow every phase in order. Do not skip phases. Ask the user before proceeding
past any phase gate.

---

## Phase 1 — Define intent

The user's intent: **$ARGUMENTS**

1. Break the intent into discrete, implementable units of work
2. For each unit, identify the specific files and code references involved
3. Present each unit to the user for validation before creating anything
4. Once confirmed, create a GitHub Issue per unit using `gh issue create`:
   - Title: concise action statement
   - Body: include code references (file:line), acceptance criteria, and links to related issues
   - Labels: add relevant labels if the repo uses them

Confirm with:
  "Created [N] issues: #[list]. Review them before I continue."

**PHASE GATE — Wait for user confirmation before proceeding.**

---

## Phase 2 — Validate issues

Start a fresh context: tell the user to run `/clear` or `/compact`.

In the new context:
1. Read every issue created in Phase 1 via `gh issue view`
2. Check each issue is accurate, correctly scoped, and not missing anything
3. Flag any issues that overlap, conflict, or are missing acceptance criteria
4. Fix any problems found

Confirm with:
  "All [N] issues validated. Ready for enrichment."

---

## Phase 3 — Enrich with parallel subagents

Spawn one subagent per issue using the Agent tool. Each subagent should:
1. Read the issue via `gh issue view`
2. Read all referenced source files
3. Perform root cause analysis (for bugs) or impact analysis (for features)
4. Add a comment to the issue with:
   - Context gathered
   - Root cause or impact analysis
   - Proposed solution approach
   - Files that will need changes
   - Risks or dependencies

Wait for all subagents to complete before continuing.

**PHASE GATE — Tell the user to `/compact` or `/clear` before implementation.**

---

## Phase 4 — Group and implement

1. Read all enriched issues via `gh issue list` and `gh issue view`
2. Group related issues that should land in the same PR
3. Present the grouping to the user for approval
4. For each group, in order:
   a. Create a feature branch from the target branch: `git checkout -b <group-branch>`
   b. Create a local `checklist.md` tracking all issues in this group:
      ```
      # PR Group: [group name]
      ## Issues
      - [ ] #123 — description
      - [ ] #456 — description
      ## Progress
      - [ ] Implementation
      - [ ] Tests written
      - [ ] Regressions fixed
      - [ ] Stale tests removed
      ```
   c. Implement fixes/features referencing each issue: `Resolves #123`
   d. Write or update tests for every change
   e. Run the test suite — fix any regressions
   f. Remove outdated or stale tests that no longer apply
   g. Update `checklist.md` as each issue is completed
   h. Commit with messages that reference the issue numbers

---

## Phase 5 — Open PR and clean up

For each completed group:
1. **Delete `checklist.md`** — run `rm checklist.md` before committing final state
2. Verify `checklist.md` is not in the staged files or git history
3. Push the branch and open a PR via `gh pr create`:
   - Title: concise summary of the group
   - Body must include:
     - Summary of changes
     - List of issues resolved: `Resolves #123, Resolves #456`
     - Test plan
4. Confirm the PR URL with the user

---

## Guardrails

- **checklist.md is ephemeral** — it must NEVER appear in any commit or PR. Delete it before the final commit in every group. If you find it staged, unstage and delete it immediately.
- Do NOT combine unrelated issues into a single PR for convenience
- Do NOT skip writing tests — every code change needs test coverage
- Do NOT modify code outside the scope of the assigned issues without flagging it
- Do NOT force-push or rewrite history on shared branches
- If the test suite fails and you cannot resolve it in 2 attempts, stop and ask the user
- Always reference issue numbers in commits and PR descriptions for traceability
