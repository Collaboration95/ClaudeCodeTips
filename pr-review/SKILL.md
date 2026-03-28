---
name: review-pr
description: Conduct a thorough, customizable review of a GitHub PR by URL. Evaluates intent, logic, implementation, consistency, and posts a structured PR review comment.
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - WebFetch
  - Agent
argument-hint: <PR-URL>
model: opus
---

# PR Review

Review a GitHub pull request by URL. The review process and output format are defined in editable resource files.

**PR URL:** $ARGUMENTS

## Phase 1: Gather PR Context

1. Parse the PR URL from `$ARGUMENTS`. Extract `owner/repo` and the PR number.
   - Valid formats: `https://github.com/{owner}/{repo}/pull/{number}`, with optional trailing `/files`, `/commits`, or fragment
   - If the URL is invalid or missing, print: `Usage: /review-pr <PR-URL>` and stop

2. Verify GitHub CLI authentication:
   ```
   gh auth status
   ```
   If not authenticated, tell the user to run `gh auth login` and stop.

3. Fetch PR metadata:
   ```
   gh pr view <URL> --json title,body,author,baseRefName,headRefName,labels,comments,reviews,files,additions,deletions,changedFiles,state
   ```

4. Fetch the comment thread in readable form:
   ```
   gh pr view <URL> --comments
   ```

5. Fetch the diff:
   - First check scope: `gh pr diff <URL> --name-only` to get the file list
   - If total `additions + deletions <= 2000`: fetch the full diff with `gh pr diff <URL>`
   - If larger: prioritize non-test, non-lock, non-generated files. Fetch diffs in batches using `gh api` for individual files if needed. Note which files were skipped.

6. If the PR is closed/merged with no open review needed, report the state and stop.

## Phase 2: Execute Review

1. Read `resources/review-process.md` for the review steps and criteria.
2. Follow each step in order, applying it to the gathered PR data.
3. For **consistency checks** (Step 3 in the default process): if you need to verify a pattern exists in files NOT in the diff, fetch those files from the repo:
   ```
   gh api repos/{owner}/{repo}/contents/{path} --jq '.content' | base64 -d
   ```
4. Record all findings with severity level, file path, and line numbers.

## Phase 3: Compile & Deliver

1. Read `resources/output-template.md` for the output structure and formatting rules.
2. Compile all findings into the template structure.
3. Omit any section that has zero findings.
4. Write the compiled review to a temporary file:
   ```
   tmpfile=$(mktemp /tmp/pr-review-XXXXXX.md)
   ```
5. Post the review to the PR:
   - If any finding is marked **critical**: `gh pr review <URL> --request-changes --body-file "$tmpfile"`
   - Otherwise: `gh pr review <URL> --comment --body-file "$tmpfile"`
6. Clean up: `rm "$tmpfile"`
7. If the review body exceeds 65,000 characters, truncate the Nits section first, then Code Quality Issues, adding a note that the full review was too large and the least critical items were trimmed.

## Error Handling

- **Invalid URL:** Print usage instructions and stop
- **Auth failure:** Tell the user to run `gh auth login` and stop
- **PR not found / permission denied:** Report the error clearly and stop
- **Empty diff:** Report that the PR has no changes to review and stop
- **gh command failure:** Report the exact error from gh and stop

## Customization

- To change **what gets reviewed**: edit `resources/review-process.md`
- To change **how findings are presented**: edit `resources/output-template.md`
- Neither change requires modifying this file
