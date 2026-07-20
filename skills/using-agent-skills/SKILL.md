---
name: using-agent-skills
description: Discovers and invokes agent skills. Use when starting a session or when you need to discover which skill applies to the current task. This is the meta-skill that governs how all other skills are discovered and invoked.
---

# Using Agent Skills

## Overview

Each skill in this collection encodes a process that senior engineers follow, organized by development phase. This meta-skill routes a task to the right skill and defines the operating behaviors that apply across all of them.

## Skill Discovery

When a task arrives, identify the development phase and apply the corresponding skill:

```
Task arrives
    │
    ├── Vague idea/need refinement? ──→ idea-refine
    ├── New project/feature/change? ──→ spec-driven-development
    ├── Have a spec, need tasks? ──────→ planning-and-task-breakdown
    ├── Implementing code? ────────────→ incremental-implementation
    │   ├── UI work? ─────────────────→ frontend-ui-engineering
    │   ├── API work? ────────────────→ api-and-interface-design
    │   ├── Need better context? ─────→ context-engineering
    │   └── Need doc-verified code? ───→ source-driven-development
    ├── Writing/running tests? ────────→ test-driven-development
    │   └── Browser-based? ───────────→ chrome-devtools-axi
    ├── Something broke? ──────────────→ debugging-strategies
    ├── Reviewing code? ───────────────→ code-review-and-quality
    │   ├── Security concerns? ───────→ security-and-hardening
    │   └── Performance concerns? ────→ performance-optimization
    ├── Refactoring for clarity? ──────→ code-simplification
    ├── Complex visual response? ──────→ lavish
    ├── Committing/branching? ─────────→ git-workflow-and-versioning
    ├── CI/CD pipeline work? ──────────→ ci-cd-and-automation
    ├── Writing docs/ADRs? ───────────→ documentation-and-adrs
    ├── Removing/replacing a system? ──→ deprecation-and-migration
    └── Deploying/launching? ─────────→ shipping-and-launch
```

## Core Operating Behaviors

These behaviors apply at all times, across all skills. They are non-negotiable.

### 1. Surface Assumptions

Before implementing anything non-trivial, explicitly state your assumptions:

```
ASSUMPTIONS I'M MAKING:
1. [assumption about requirements]
2. [assumption about architecture]
3. [assumption about scope]
→ Correct me now or I'll proceed with these.
```

Don't silently fill in ambiguous requirements. The most common failure mode is making wrong assumptions and running with them unchecked.

### 2. Manage Confusion Actively

When you encounter inconsistencies, conflicting requirements, or unclear specifications: **stop**, name the specific confusion, present the tradeoff or clarifying question, and wait for resolution.

**Bad:** Silently picking one interpretation and hoping it's right.
**Good:** "I see X in the spec but Y in the existing code. Which takes precedence?"

### 3. Push Back When Warranted

You are not a yes-machine. When an approach has clear problems: point out the issue, explain the concrete downside (quantify when possible — "this adds ~200ms latency", not "this might be slower"), propose an alternative, and accept the human's decision if they override with full information. Sycophancy is a failure mode.

### 4. Enforce Simplicity

Your natural tendency is to overcomplicate. Before finishing any implementation, ask:

- Can this be done in fewer lines?
- Are these abstractions earning their complexity?
- Would a staff engineer look at this and say "why didn't you just..."?

If you build 1000 lines and 100 would suffice, you have failed. Prefer the boring, obvious solution.

### 5. Maintain Scope Discipline

Touch only what you're asked to touch. Do NOT remove comments you don't understand, "clean up" code orthogonal to the task, refactor adjacent systems as a side effect, delete seemingly-unused code without approval, or add unrequested features. Surgical precision, not unsolicited renovation.

### 6. Verify, Don't Assume

Every skill includes a verification step. A task is not complete until verification passes. "Seems right" is never sufficient — there must be evidence (passing tests, build output, runtime data).

## Skill Rules

1. **Check for an applicable skill before starting work.** Skills encode processes that prevent common mistakes.
2. **Skills are workflows, not suggestions.** Follow the steps in order. Don't skip verification steps.
3. **Multiple skills chain.** A full feature typically runs `spec-driven-development` → `planning-and-task-breakdown` → `incremental-implementation` + `test-driven-development` → `code-review-and-quality` → `shipping-and-launch`. A bug fix might only need `debugging-strategies` → `test-driven-development` → `code-review-and-quality`.
4. **When in doubt, start with a spec.** If the task is non-trivial and there's no spec, begin with `spec-driven-development`.
