---
allowed-tools:
  - Bash(gh issue view:*)
  - Bash(gh search:*)
  - Bash(gh issue list:*)
  - Bash(gh pr comment:*)
  - Bash(gh pr diff:*)
  - Bash(gh pr view:*)
  - Bash(gh pr list:*)
  - Bash(git log:*)
  - Bash(git blame:*)
  - Bash(git show:*)
  - Read
  - Glob
  - Grep
  - Task
description: Intelligent code review with smart re-review support for new commits
---

# Smart Code Review

Provide a code review for the given pull request with intelligent re-review support.

## Instructions

Follow these steps precisely:

### Step 1: Smart Eligibility Check

Use a Haiku agent to check if the pull request is eligible for review and determine the review mode.

The agent should:
1. Use `gh pr view <PR> --json state,isDraft,author,title,labels,comments,commits` to get PR details
2. Check if PR is closed, draft, or automated (skip if any are true)
3. Look for existing Claude review comments (author login contains "claude" or body contains "Generated with Claude Code")
4. If an existing review exists:
   a. Extract the review timestamp
   b. Get commits after that timestamp using: `gh pr view <PR> --json commits`
   c. If new commits exist since review: Return `mode: "re-review"` with the previous review body
   d. If no new commits: Return `mode: "skip"` (already reviewed, no changes)
5. If no existing review: Return `mode: "full"`

The agent should return a JSON-like response:
- `{ mode: "skip", reason: "..." }` - Do not proceed
- `{ mode: "full" }` - Run full review
- `{ mode: "re-review", previousReview: "...", previousCommitSha: "..." }` - Run re-review

If mode is "skip", do not proceed with the review.

### Step 2: Gather CLAUDE.md Files

Use a Haiku agent to find all relevant CLAUDE.md files:
1. Root CLAUDE.md (if exists)
2. CLAUDE.md files in directories modified by the PR

Return a list of file paths.

### Step 3: Summarize the PR

Use a Haiku agent to view the PR diff and return a brief summary of the changes.

### Step 4: Run Review Agents

#### If mode is "full" (Initial Review):

Launch 5 parallel Sonnet agents to independently review the change:

1. **Agent #1 - CLAUDE.md Compliance**: Audit changes for compliance with CLAUDE.md guidelines. Focus on rules that are explicitly stated.

2. **Agent #2 - Bug Scanner**: Read the file changes and scan for obvious bugs. Focus only on the changes themselves, not surrounding code. Look for large bugs, avoid nitpicks. Ignore likely false positives.

3. **Agent #3 - Git History Context**: Read git blame and history of modified code. Identify any bugs in light of that historical context.

4. **Agent #4 - Previous PR Comments**: Read previous PRs that touched these files. Check for comments that may also apply here.

5. **Agent #5 - Code Comments Compliance**: Read code comments in modified files. Ensure changes comply with any guidance in those comments.

Each agent should return a list of issues with:
- Issue description
- File and line reference
- Reason flagged (CLAUDE.md, bug, historical context, etc.)

#### If mode is "re-review" (Follow-Up Review):

Launch 5 parallel agents:

1. **Agent #1 - Previous Issues Verification (Sonnet)**: Parse the previous review to extract issues. For each issue, check if it was addressed in the new commits. Return status for each: FIXED or STILL_PRESENT with current location.

2. **Agent #2-5 - New Issues Scan (Sonnet)**: Scan ONLY the new commits (not the entire PR) for new issues. Use the same criteria as the full review agents above, but focus only on code added/modified since the previous review.

### Step 5: Score Issues

For each issue found in Step 4, launch a parallel Haiku agent to score confidence (0-100):

Scoring rubric (provide verbatim to agents):
- **0**: False positive that doesn't stand up to scrutiny, or pre-existing issue
- **25**: Might be real but could be false positive. Stylistic issues not explicitly in CLAUDE.md
- **50**: Verified real issue, but might be a nitpick or rare in practice. Not very important relative to PR
- **75**: Double-checked and very likely real. Will be hit in practice. Existing approach is insufficient. Very important or directly mentioned in CLAUDE.md
- **100**: Absolutely certain. Confirmed real issue that will happen frequently. Evidence directly confirms this

For CLAUDE.md issues, the agent must verify the CLAUDE.md actually calls out that specific issue.

### Step 6: Filter Issues

Filter out any issues with a score less than 80.

If no issues meet this threshold and mode is "full", proceed to Step 7 with "no issues found".
If no issues meet this threshold and mode is "re-review", proceed to Step 7 with status of previous issues only.

### Step 7: Re-Check Eligibility

Use a Haiku agent to repeat the eligibility check from Step 1, ensuring the PR is still open and eligible.

### Step 8: Post Comment

Use `gh pr comment <PR> --body "..."` to post the review.

#### Initial Review Format (mode: "full"):

If issues found:
```
### Code review

Found N issues:

1. <brief description> (CLAUDE.md says "<relevant quote>")

<link to file with full SHA and line range, e.g., https://github.com/owner/repo/blob/abc123def/path/file.ts#L10-L15>

2. <brief description> (bug due to <reason>)

<link>

Generated with [Claude Code](https://claude.ai/code)

<sub>- If this code review was useful, react with thumbs up. Otherwise, react with thumbs down.</sub>
```

If no issues:
```
### Code review

No issues found. Checked for bugs and CLAUDE.md compliance.

Generated with [Claude Code](https://claude.ai/code)
```

#### Follow-Up Review Format (mode: "re-review"):

```
### Code review (follow-up)

**Previous Issues** (from review at <previous commit SHA>):
- [FIXED] <issue description>
- [STILL PRESENT] <issue description>
  <link to current location>

**New Issues Found** (in commits since last review):
1. <new issue description>
<link>

Summary: X/Y previous issues fixed, Z new issues found.

Generated with [Claude Code](https://claude.ai/code)

<sub>- If this code review was useful, react with thumbs up. Otherwise, react with thumbs down.</sub>
```

If all previous issues fixed and no new issues:
```
### Code review (follow-up)

**Previous Issues** (from review at <previous commit SHA>):
- [FIXED] <issue 1>
- [FIXED] <issue 2>

All previous issues have been addressed. No new issues found.

Generated with [Claude Code](https://claude.ai/code)
```

## Important Notes

- Keep output brief, avoid emojis
- Link and cite relevant code, files, and URLs
- Use full git SHA in links (not HEAD or branch names)
- Line range format: `#L<start>-L<end>` with 1 line of context before/after
- Repo name must match the repo being reviewed

## False Positives to Avoid

- Pre-existing issues (not introduced by PR)
- Code that looks like a bug but is correct
- Pedantic nitpicks a senior engineer wouldn't flag
- Issues a linter/typechecker would catch
- General quality issues unless required by CLAUDE.md
- Issues silenced by lint ignore comments
- Intentional functionality changes
- Issues on lines not modified in the PR
