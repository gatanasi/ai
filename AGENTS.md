# AGENTS.md

Operating standard for AI agents (Claude, Codex, Gemini, OpenCode, ...) working in this repository.
Priorities: (1) Correctness, (2) Root-cause resolution, (3) Maintainability.

## Skills

Skills live in `skills/<name>/SKILL.md`. If a task matches a skill — even partially — invoke it and follow it exactly. Never implement directly when a skill applies. When the match is unclear, consult `using-agent-skills` (the router). Multiple skills often chain: complete the required upstream steps (spec, plan) before implementing.

| Phase | Trigger | Skill(s) |
|---|---|---|
| DEFINE | Vague idea needing refinement | `idea-refine` |
| DEFINE | New feature or significant change | `spec-driven-development` |
| PLAN | Spec exists, needs breakdown | `planning-and-task-breakdown` |
| BUILD | Implementing anything | `incremental-implementation` + `test-driven-development` |
| BUILD | UI work / API design / framework code | `frontend-ui-engineering` / `api-and-interface-design` / `source-driven-development` |
| VERIFY | Bug, failure, unexpected behavior | `debugging-strategies` |
| VERIFY | Browser-based behavior | `browser-testing-with-devtools` |
| REVIEW | Code review | `code-review-and-quality` |
| REVIEW | Security or performance concern | `security-and-hardening` / `performance-optimization` |
| SHIP | Commit, refactor, CI, deploy | `git-workflow-and-versioning`, `code-simplification`, `ci-cd-and-automation`, `shipping-and-launch` |

## Operating Rules

1. **Plan first.** For any non-trivial task (3+ steps or an architectural decision), plan in read-only mode before writing code — use plan mode if your tool has one. If work goes sideways, stop and re-plan; don't keep pushing.
2. **Verify before done.** Never mark a task complete without evidence: tests, logs, diffs, or runtime checks. Ask: "Would a staff engineer approve this?"
3. **Demand elegance, balanced.** For non-trivial changes, ask "is there a more elegant way?" If a fix feels hacky, re-implement it knowing everything you know now. Skip this for simple, obvious fixes — don't over-engineer.
4. **Fix bugs autonomously.** Given a bug report, logs, or failing CI: reproduce, find the root cause, and fix it without asking for hand-holding. No temporary patches.
5. **Use subagents for research.** When your tool supports them, offload exploration and parallel analysis to subagents (one focused task each) to keep the main context clean.
6. **Simplicity first.** Make every change as simple as possible; impact minimal code.

## PR Review Protocol

When asked to process PR comments: fetch inline comments and review threads (`gh` CLI or GitHub MCP); triage each as accept / adapt / decline against repo best practices — don't blindly comply with bad suggestions, propose the better alternative; apply fixes holistically, not line-by-line; then resolve the threads and report what was fixed and what needs reviewer input.

## Task Management

1. Write the plan to `tasks/todo.md` as checkable items; confirm it before implementing.
2. Mark items complete as you go and summarize changes at each step.
3. When done, add a review section to `tasks/todo.md`.
4. After any correction from the user, record the pattern in `tasks/lessons.md` and review that file at session start.
