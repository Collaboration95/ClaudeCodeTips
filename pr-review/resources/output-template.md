# PR Review Output Template

Defines the structure, sections, and formatting of the review output.
Edit this file to customize the PR comment format.

## Structure

The review comment must follow this structure exactly.
**Omit any section that has zero findings.**

## Heading

Start the review with:

```
## PR Review: #{pr_number} — {pr_title}
```

## Sections

### Section: Logic Issues

**Heading:** `### Logic Issues`
**What goes here:** Flaws in the overall approach or strategy. These are problems with WHAT the PR is doing, not HOW the code is written. Architectural mistakes, wrong abstractions, solving the wrong problem.

**Item format:**
```
- **[critical|major]** {description}
  - Context: {why this is a problem}
  - Suggestion: {what to do instead}
```

### Section: Code-Level Logic Issues

**Heading:** `### Code-Level Logic Issues`
**What goes here:** Logic bugs in the implementation — the approach is correct but the execution is wrong. Off-by-one errors, wrong conditions, missing branches, incorrect state transitions.

**Item format:**
```
- **{file}:{line}** — {description}
  - Expected: {what should happen}
  - Actual: {what will happen}
  - Fix: {concrete suggestion}
```

### Section: Code Quality Issues

**Heading:** `### Code Quality Issues`
**What goes here:** Non-logic problems: error handling gaps, missing validation, inefficiency, inconsistency across tools/fetchers, security issues.

**Item format:**
```
- **{file}:{line}** — {description} `{category}`
  - Impact: {what could go wrong}
  - Fix: {concrete suggestion}
```

**Categories:** `error-handling`, `efficiency`, `consistency`, `security`, `validation`, `testing`

### Section: Nits

**Heading:** `### Nits`
**What goes here:** Minor, non-blocking observations. Naming inconsistencies, minor style issues that don't affect correctness. Keep this section short.

**Item format:**
```
- **{file}:{line}** — {description}
```

## Severity Levels

- **critical** — Must fix before merge. Broken logic, data loss risk, security vulnerability, silent failure in production.
- **major** — Should fix before merge. Incorrect behavior in edge cases, missing error handling for likely scenarios, consistency violations across tools.
- **minor** — Non-blocking. Suboptimal patterns, missing tests for unlikely paths, naming inconsistencies.

## Rules

- No "Strengths", "What looks good", or "Positive observations" sections — this review is issues-only
- No preamble, greeting, or sign-off — start directly with the heading
- If a section has zero findings, omit it entirely (do not include an empty section)
- Every item in Code-Level Logic Issues and Code Quality Issues must reference a specific file and line number
- Logic Issues may be architectural and not tied to a specific line — that is acceptable
- Keep descriptions concise — one sentence for the issue, one for the fix
- Use inline code blocks for specific code references (variable names, function calls, etc.)

## PR Review Action

- If ANY item is marked **critical**: post with `--request-changes`
- Otherwise: post with `--comment`
