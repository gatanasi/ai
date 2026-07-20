---
name: code-review-and-quality
description: Reviews code changes effectively. Use when reviewing pull requests or diffs, responding to review feedback, or establishing review standards. Catches bugs early and keeps feedback constructive and actionable.
---

# Code Review and Quality

## Overview

A review's job is to catch problems early, keep the codebase maintainable, and spread knowledge — not to gatekeep or rewrite the change to your taste. This skill defines the review *process* and feedback craft. For the five-dimension evaluation framework and the structured report format, use the `agents/code-reviewer.agent.md` persona (run it as a subagent when available); for deep security review, `security-and-hardening` or the `agents/security-auditor.agent.md` persona.

## When to Use

- Reviewing a pull request or local diff before merge
- Responding to review comments on your own change (triage: accept / adapt / decline)
- Establishing review standards for a project

## Review Scope

**Review:** logic correctness and edge cases, security, performance, test coverage and quality, error handling, API design and naming, architectural fit.

**Don't review manually:** formatting, import order, lint violations — automate these (Prettier, ESLint, Black); a machine should comment on style so humans can comment on substance.

## The Review Process

### Phase 1: Context

1. Read the PR description, linked issue, and spec — know what the change is *supposed* to do.
2. Check CI status and the diff size. Over ~400 lines, ask to split (see `git-workflow-and-versioning` for sizing).
3. **Read the tests first** — they reveal intent and coverage gaps faster than the implementation.

### Phase 2: High-Level Pass

- Does the solution fit the problem? Is there a simpler approach?
- Is it consistent with existing patterns, or does it introduce a new one (and is that justified)?
- Are files in the right places? Is the change cohesive, or several changes in a trench coat?

### Phase 3: Line-by-Line Pass

- Correctness: edge cases (null/empty/boundary), off-by-one, race conditions, error paths
- Security: input validation, parameterized queries, authz on every endpoint, no secrets, output encoding
- Performance: N+1 queries, unbounded fetches, missing pagination, blocking I/O in hot paths
- Maintainability: names, single-purpose functions, magic numbers, dead code
- Dependencies: does the existing stack already solve this? Check size, maintenance, vulnerabilities, license. Every new dependency is a liability.

### Phase 4: Summary and Decision

Summarize key concerns, name what was done well (at least one specific thing), and give a clear verdict: approve / comment / request changes. Never approve with unresolved critical issues.

## Feedback Craft

**Label severity** so the author knows what blocks and what doesn't:

- `[blocking]` — must fix before merge (bug, security, data loss)
- `[important]` — should fix; discuss if you disagree
- `[nit]` — preference, not blocking
- `[question]` / `[praise]` — no action required

**Be specific and actionable.** Every blocking/important comment names the problem, the consequence, and a concrete fix:

```
❌ "This is wrong."
✅ "[blocking] This can race when two requests update the same task —
   last write wins and drops fields. Consider optimistic locking via
   the updatedAt check we use in TaskService.update."
```

**Ask questions to prompt thinking** ("What happens if `items` is empty?") and **suggest rather than command** ("Would extracting this into a shared utility make sense? It appears in 3 places."). Critique the code, not the person.

**Handling disagreement:** seek to understand the author's reasoning first; acknowledge valid points; bring data (benchmark, repro) over opinion; escalate to a third reviewer if stuck; and let go of non-critical preferences — perfection is the enemy of progress. The same triage applies when *receiving* review comments: accept, adapt, or decline with rationale — don't blindly comply with bad suggestions.

## Review Hygiene

- Review promptly (within 24h) — stale PRs rot and block others
- 60 minutes max per sitting; effectiveness drops sharply after
- Same standards for every author; no rubber-stamping, no ghosting after requesting changes
- Watch for scope creep in your own comments ("while you're at it..." is a new task, not a review comment)

## Red Flags

- Approving without reading the tests (or with no tests present for behavior changes)
- Review threads full of formatting nits (automate them instead)
- Feedback without severity labels — author can't tell what blocks merge
- "Request changes" verdicts with vague, unactionable comments
- Large PRs reviewed as a skim because splitting felt like too much work
- Blindly applying reviewer suggestions that conflict with repo conventions

## Verification

After completing a review (or a review-response round):

- [ ] All blocking issues resolved; important issues resolved or explicitly deferred with justification
- [ ] Tests pass and cover the changed behavior
- [ ] Build succeeds
- [ ] Verdict and rationale are recorded on the PR
- [ ] Resolved threads closed; remaining questions assigned to someone
