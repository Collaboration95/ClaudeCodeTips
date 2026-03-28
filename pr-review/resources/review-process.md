# PR Review Process

Defines the review steps, criteria, and evaluation order.
Edit this file to customize what gets reviewed and how.

## Step 1: Understand Intent

- Read the PR title, description, and any linked issues
- Read existing review comments and author responses
- Identify the stated goal: bug fix, feature, refactor, config change, dependency update, etc.
- Summarize the intent in 1-2 sentences (internal working note — do NOT include in output)

## Step 2: Evaluate Logic Validity

Evaluate the PR's approach independently of the code quality:

- Does the approach make sense for the stated goal?
- Are there logical flaws in the overall strategy?
- Are there edge cases the approach fails to handle?
- Does the PR solve the actual problem, or does it address a symptom?
- Are there simpler alternatives that achieve the same outcome?
- If the logic is fundamentally flawed, stop here and report — do not proceed to code review

## Step 3: Review Code Implementation

For each changed file, evaluate the following areas:

### Correctness
- Off-by-one errors, null/undefined handling, type mismatches
- Race conditions, deadlocks, resource leaks
- Incorrect API usage, wrong method signatures
- Missing return statements, unreachable code

### Efficiency
- Unnecessary iterations, redundant computations
- Missing caching where appropriate
- N+1 queries, unbounded loops, excessive memory allocation
- Blocking calls where async is expected

### Error Handling
- Silent failures, empty catch blocks, swallowed errors
- Missing error propagation, inadequate logging
- User-facing errors that leak internals
- Missing retries or fallbacks for external service calls

### Consistency
- If one tool/fetcher follows a pattern (e.g., filtering only error and warning logs), ALL similar tools/fetchers must follow the same pattern
- Naming conventions across files
- Import patterns and module structure consistency
- Configuration patterns (env vars, defaults) applied uniformly

### Security
- Injection vulnerabilities (SQL, command, template)
- Secrets or credentials in code
- Missing input validation or sanitization
- Overly permissive permissions or CORS settings

## Step 4: Cross-Cutting Concerns

- Are tests added or updated for the changes?
- Does the PR break any existing interfaces or contracts?
- Are there documentation gaps for public APIs or config changes?
- Backward compatibility: can this be deployed without a migration step?
- Are there any hardcoded values that should be configurable?

## Guidelines

- Do NOT comment on style preferences unless they clearly violate established project conventions
- Do NOT praise code — focus exclusively on issues and risks
- Every issue must include: file path, line reference (where applicable), what is wrong, and why it matters
- Distinguish between "must fix before merge" (critical) and "should fix but non-blocking" (minor)
- If you find zero issues, state that clearly — do not manufacture concerns
- When checking consistency, fetch non-diff files from the repo if needed to verify patterns
