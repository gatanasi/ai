# AI Engineering Operating System

This is my personal AI engineering operating system: the rules, skills, and
specialized agent prompts I use to make AI-assisted software work disciplined,
reviewable, and production-minded.

## What This Demonstrates

- **Spec-first execution:** ambiguous requests are turned into explicit
  objectives, constraints, success criteria, and boundaries before code changes.
- **Plan-before-build discipline:** non-trivial work is broken into ordered,
  verifiable tasks instead of being implemented as one large opaque change.
- **Incremental delivery:** changes are sliced so each step can be reviewed,
  tested, and rolled back independently.
- **Verification bias:** agents are expected to prove behavior with tests,
  logs, browser evidence, diffs, or runtime checks before claiming completion.
- **Security and review habits:** security, maintainability, and code-review
  standards are encoded directly into reusable skills and agent personas.
- **Agent specialization:** focused reviewer, security, testing, and domain
  skills keep work structured instead of relying on one generic prompt.

## Repository Map

| Path | Purpose |
| --- | --- |
| `AGENTS.md` | Top-level operating standard for agents working in a repo. |
| `agents/` | Focused agent personas for code review, security auditing, and testing. |
| `skills/` | Reusable workflows for planning, implementation, debugging, testing, UI work, security, shipping, and related engineering tasks. |

## How I Use It

When I start a task with an AI agent, this repo defines the default posture:

1. Identify the intent and select the relevant skill.
2. Ground the work in the real repository, logs, tests, or runtime evidence.
3. Write or refine the spec and implementation plan when the task is
   non-trivial.
4. Implement in small slices, keeping the diff easy to inspect.
5. Verify the result with the strongest practical evidence available.
6. Review the work critically before treating it as complete.

The goal is not to make AI produce more code. The goal is to make AI operate
inside the same standards I would expect from a careful engineer: understand the
problem, preserve context, make small defensible changes, and prove the outcome.

## Design Principles

- **Correctness over speed:** fast output is not useful if the root cause is
  wrong or the fix cannot be trusted.
- **Evidence over speculation:** logs, tests, source documentation, and runtime
  inspection beat plausible guesses.
- **Maintainability over cleverness:** the best AI-assisted change should still
  be simple for a human engineer to review later.
- **Critical review over obedience:** reviewer comments, tool output, and agent
  suggestions are evaluated for correctness before being applied.
- **Personal workflow over product polish:** these files reflect how I work;
  they are intentionally not wrapped as a public CLI, package, or marketplace.

## Reading Path

If you are reviewing this as a portfolio artifact, start with:

1. `AGENTS.md` for the overall standard.
2. `skills/spec-driven-development/SKILL.md` for how vague work becomes a
   concrete spec.
3. `skills/planning-and-task-breakdown/SKILL.md` for implementation planning.
4. `skills/incremental-implementation/SKILL.md` and
   `skills/test-driven-development/SKILL.md` for build and verification habits.
5. `agents/code-reviewer.agent.md`, `agents/security-auditor.agent.md`, and
   `agents/test-engineer.agent.md` for specialized review roles.
