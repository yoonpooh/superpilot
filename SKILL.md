---
name: superpilot
description: "Use when handling non-trivial work in an existing repository and the agent should move from collaborative design into autonomous implementation with spec, plan, TDD, optional subagents, harsh diff review, and evidence-based verification."
---

# Superpilot

## Mission

Use this skill for non-trivial work in an existing repository when the job should move from collaborative design into autonomous implementation without depending on any external workflow skill pack.

This skill is self-contained:

- do not rely on external workflow skills to define the core process
- do not scan for other workflow skills before each stage
- use runtime tools, shell commands, git, and subagents directly when needed
- do not commit, push, or open PRs unless the user explicitly asks

## Workflow

For non-trivial work, follow this path:

1. Explore repository context and local guardrails
2. Clarify only what is materially ambiguous
3. Collaborate with the user on the design
4. Write a spec
5. Write a plan
6. Investigate root cause first when debugging is needed
7. Execute autonomously
8. Run harsh review-and-patch loops on the diff
9. Verify with fresh evidence
10. Deliver a completion summary

Once the request is clear enough to produce a correct spec, treat the original user request as authorization to continue through planning and implementation. Do not introduce approval gates unless a safety gate triggers.

If the user explicitly asks for review-only output on an existing diff, branch, commit, or pull request, switch to review-only mode:

- skip spec, plan, and implementation
- inspect the diff and return findings only
- do not patch unless the user explicitly asks for fixes

## Hard Rules

- Treat `AGENTS.md` in the active repository as a strong local rule set.
- For non-trivial work, always produce both a spec and a plan.
- Review-only requests are an explicit exception to the spec-and-plan rule.
- When the task involves a bug, test failure, build failure, or unexpected behavior, investigate root cause before proposing fixes.
- Use TDD for feature work, bug fixes, refactoring, and behavior changes when there is a real executable test surface.
- Choose the execution strategy yourself. Do not ask the user to choose direct execution vs subagent execution.
- Use subagents only when task independence is clear and write scopes do not overlap.
- Final integration, final review, and final verification always belong to the main agent.
- Review the diff, not the untouched codebase, and keep looping until actionable findings reach zero.
- Treat review findings as internal work items unless the user explicitly asks for a review-only report.
- Never claim completion without fresh verification evidence.
- Do not do speculative refactoring or unrelated cleanup.
- Do not commit, push, or publish unless the user explicitly asks.

## Trivial Exception

Only truly trivial work may skip the full workflow. A task is trivial only when all of these are true:

- single-line or few-character change
- zero ambiguity about what to change
- zero side-effect risk
- no design decision involved

Trivial work may skip the spec and plan, but it must still respect local guardrails, verification, and scope discipline.

## Storage

Write specs and plans under:

```text
~/.superpilot/docs/<repo-name>/
├── specs/
└── plans/
```

Use the git root basename as `<repo-name>`. Name files as:

```text
YYYY-MM-DD-<short-slug>.md
```

Use short slugs that describe the task itself, not the branch name.

## References

Load the relevant reference file for the current stage:

- [references/spec.md](references/spec.md): collaborative design, spec writing, execution transition
- [references/plan.md](references/plan.md): implementation planning and subagent split rules
- [references/debugging.md](references/debugging.md): root-cause investigation before fixes
- [references/implementation.md](references/implementation.md): TDD, execution discipline, blocker handling
- [references/review.md](references/review.md): harsh diff review procedure and checklist
- [references/verification.md](references/verification.md): evidence-based verification and completion rules

The main file defines the contract. The reference files define how to execute each stage.

## Safety Gates

Stop and ask the user only when:

- an action is destructive or irreversible
- credentials, secrets, or access are missing
- requirements are contradictory or newly unsafe

Do not stop for routine execution choices, planning choices, or implementation sequencing.

## Non-Goals

This skill is not for:

- greenfield project creation
- release or deployment orchestration by default
- commit and PR workflows by default
- replacing repository-specific rules in `AGENTS.md`
