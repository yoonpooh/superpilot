---
name: superpilot
description: "Use when handling non-trivial work in an existing repository, or when performing a review-only audit of an existing diff, branch, commit, or pull request."
---

# Superpilot

## Mission

Use this skill for non-trivial work in an existing repository when the job should move from collaborative design into autonomous implementation without depending on any external workflow skill pack.

This skill is self-contained:

- do not rely on external workflow skills to define the core process
- do not scan for other workflow skills before each stage
- use runtime tools, shell commands, git, and subagents directly when needed
- do not commit, push, or open PRs unless the user explicitly asks

The default objective is zero-intervention completion:

- drive the task from first read to verified handoff without waiting for routine approvals
- research before guessing when external facts, library behavior, or domain rules affect the solution
- preserve the full requested scope from user request -> spec -> plan -> implementation -> review -> verification
- choose the execution topology yourself: in-place, isolated branch/worktree, direct execution, or bounded fresh subagents
- actively evaluate bounded fresh subagents for every non-trivial task, and use them when independent work can run safely in parallel
- prove real readiness with the strongest relevant checks, including user-flow, DX, migration, and security checks when applicable

For non-implementation requests such as explanation, analysis, advice, critique, or explicit "do not edit/work" requests, answer only within the requested scope. Do not transition into planning or implementation unless the user explicitly asks for changes.

Before entering the full workflow, classify the request. Low-risk, tightly scoped tasks where the requested change is explicit, localized, and does not require design judgment use Micro Task Mode instead of the full workflow.

Before writing a spec for the full workflow, confirm the requirements. Every must-have requirement must be either explicitly confirmed by the user or directly proven by repository evidence. Do not start the spec from inferred, likely, or assumed user requirements.

## When to Use

Use `superpilot` when working in an existing repository and the task needs disciplined execution, such as:

- bug fixes
- feature work
- refactoring tied to a real request
- config, CI, or workflow changes
- test additions or test repair
- review-only audits of an existing diff, branch, commit, or pull request

```dot
digraph superpilot_when {
    "Existing repository task?" [shape=diamond];
    "Review-only request on existing diff/branch/commit/PR?" [shape=diamond];
    "Non-trivial implementation or bugfix?" [shape=diamond];
    "Use superpilot\n(review-only mode)" [shape=box];
    "Use superpilot\n(full workflow)" [shape=box];
    "Do not use superpilot" [shape=box];

    "Existing repository task?" -> "Review-only request on existing diff/branch/commit/PR?" [label="yes"];
    "Existing repository task?" -> "Do not use superpilot" [label="no"];
    "Review-only request on existing diff/branch/commit/PR?" -> "Use superpilot\n(review-only mode)" [label="yes"];
    "Review-only request on existing diff/branch/commit/PR?" -> "Non-trivial implementation or bugfix?" [label="no"];
    "Non-trivial implementation or bugfix?" -> "Use superpilot\n(full workflow)" [label="yes"];
    "Non-trivial implementation or bugfix?" -> "Do not use superpilot" [label="no"];
}
```

## When Not to Use

Do not use `superpilot` for:

- greenfield project creation
- simple factual questions or code explanation with no change requested
- release or deployment orchestration by default
- commit or PR workflow by default

## Workflow

For non-trivial work that does not qualify for Micro Task Mode, follow this path:

1. Explore repository context, local guardrails, current baseline, and whether isolated execution is needed
2. Research external facts, library behavior, domain constraints, and prior art when uncertainty or risk is real
3. Clarify requirements until every must-have requirement is confirmed or code-proven
4. Collaborate with the user on the design and pressure-test framing, scope, UX, DX, and risk
5. If the task involves a bug, failing test, build failure, or unexpected behavior, investigate and capture the root cause first — load [references/debugging.md](references/debugging.md) before writing the spec or plan
6. Write a spec — load [references/spec.md](references/spec.md) first
7. Write a plan — load [references/plan.md](references/plan.md) first, and include the proven root cause when debugging was required
8. Load [references/implementation.md](references/implementation.md), then execute autonomously following its TDD, isolation, and execution rules
9. **Mandatory transition**: when implementation reaches GREEN, stop and load [references/review.md](references/review.md) before any other action. GREEN tests do not authorize completion — only a zero-findings review loop does. **Within the full workflow, this transition is never exempt — not by mechanical simplicity, not by test confidence.**
10. Run harsh review-and-patch loops on the diff following the loaded review procedure, including requirement-preservation and drift checks
11. Verify with fresh evidence — load [references/verification.md](references/verification.md) first
12. Deliver a completion summary and capture reusable learnings when they materially reduce future rework

Each full-workflow stage must read the linked reference file before starting work. Skipping a reference load is not allowed in the full workflow — the reference defines how to execute the stage, not just what the stage is about.

### Stage transition markers

Each stage ends with a completion marker that confirms the stage was properly completed before moving on. These markers serve as handoff contracts between stages.

| Stage | Exit marker | Meaning |
|-------|-----------|---------|
| Spec | `## SPEC COMPLETE` | Spec written, self-reviewed, path provided |
| Plan | `## PLAN COMPLETE` | Plan written, self-reviewed, execution mode chosen |
| Debugging | `## ROOT CAUSE CONFIRMED` | Root cause proven by evidence, captured before spec/plan when debugging is required |
| Implementation | `## IMPLEMENTATION GREEN` | All slices green, ready for review |
| Review | `## REVIEW COMPLETE — 0 FINDINGS` | Zero findings on fresh final pass |
| Verification | `## VERIFICATION PASSED` | Fresh evidence supports all claims |

When transitioning between stages, confirm the previous stage's exit marker condition is met. Do not advance if the exit condition is not satisfied.

For agent recovery interruptions, the current stage's marker remains unsatisfied. After recovery, resume from the incomplete stage, not the next one.

For explicit implementation requests, once the requirements are confirmed or code-proven and the request is clear enough to produce a correct spec, treat the original user request as authorization to continue through planning and implementation. Do not introduce approval gates unless a safety gate triggers.

If the user explicitly asks for review-only output on an existing diff, branch, commit, or pull request, switch to review-only mode:

- skip spec, plan, and implementation
- inspect the diff and return findings only
- do not patch unless the user explicitly asks for fixes
- still apply the same review standard, but treat findings as report output rather than internal patch work

## Hard Rules

- Treat repository-local agent instruction files such as `AGENTS.md` and `CLAUDE.md` as strong local rule sets.
- Optimize for zero-intervention completion. Ask the user only when proceeding would materially risk building the wrong thing or doing something unsafe.
- Micro Task Mode is an explicit exception to the full workflow's spec, plan, TDD, subagent-evaluation, stage-reference, and review-loop requirements.
- For non-trivial work, always produce both a spec and a plan.
- Do not write a full-workflow spec while any must-have requirement remains inferred, likely, or assumed. Ask a concrete question first.
- Review-only requests are an explicit exception to the spec-and-plan rule.
- Research before deciding when external APIs, third-party libraries, current product facts, or domain rules could change the correct implementation.
- Maintain a requirement ledger from the original request through final verification. Do not silently reduce scope, drop requested outcomes, or “simplify” by omission.
- When the task involves a bug, test failure, build failure, or unexpected behavior, investigate root cause before proposing fixes.
- Use TDD for feature work, bug fixes, refactoring, and behavior changes when there is a real executable test surface.
- Choose the execution strategy yourself. Do not ask the user to choose direct execution vs subagent execution.
- Prefer isolated execution (clean branch or worktree) when the tree is dirty, parallel work is active, or the task is risky enough that comparison/rollback clarity matters.
- For non-trivial work, subagent use must be an explicit planning decision, not an afterthought. The plan must say `direct`, `subagent`, or `mixed`, name any subagent candidates, and explain why direct-only execution is appropriate when no subagent is used.
- Prefer subagent or mixed execution when there are two or more independent investigation, implementation, test, or review slices with disjoint ownership and the main agent can do useful non-overlapping work while they run.
- Use subagents only when task independence is clear and write scopes do not overlap.
- Use fresh subagent context per independent task unless there is a strong reason to reuse prior context.
- If a task is large, spans multiple subsystems, or has parallelizable review/verification risk, default to looking for at least one bounded subagent slice before choosing direct-only execution.
- Final integration, final review, and final verification always belong to the main agent.
- Review the diff, not the untouched codebase, and keep looping until actionable findings reach zero.
- Every review pass must run mandatory trace activities, walk through every checklist item, and answer every adversarial question against the diff. A review pass that skips any of these is not a valid pass and does not count toward the zero-findings exit condition.
- Mandatory trace activities require tracing actual code paths (failure paths, state consistency, access control, input boundaries), not checking boxes. If the diff has an external API call, trace what happens when it fails. If the diff has a new endpoint, compare its access control to existing endpoints. If the diff updates state in two places, trace what happens when one update fails.
- For request-entry, redirect, auth, middleware, canonicalization, token, cookie, query-param, or session changes, treat execution order and preservation of global invariants as first-class review concerns, not implicit assumptions.
- For schema, contract, codegen, fixture, or documentation-coupled changes, treat missing paired updates as in-scope defects, not optional cleanup.
- For user-facing, onboarding, or developer-workflow changes, code inspection alone is not enough — verify the real flow with the strongest available walkthrough, browser, or smoke path.
- For security-sensitive work, make the threat model explicit in review and verification instead of assuming normal tests cover it.
- Treat review findings as internal work items unless the user explicitly asks for a review-only report.
- For implementation tasks, do not call the work complete until the internal review loop has already absorbed and patched all in-scope actionable findings.
- Do not call the task complete until a fresh final review pass after the last patch also returns zero actionable findings.
- If a subsequent "code review" request on the same finished diff would find real in-scope issues, the original review loop failed. The review must be thorough enough that a second independent review should not uncover anything new.
- Do not treat passing tests, a green build, or a reproduced fix as a substitute for the final review loop. Verification does not replace review.
- If you cannot point to fresh final review evidence for the current diff, the task is not complete yet.
- Never claim completion without fresh verification evidence.
- Do not do speculative refactoring or unrelated cleanup.
- Do not commit, push, or publish unless the user explicitly asks.

## Micro Task Mode

Use Micro Task Mode for low-risk, tightly scoped tasks where the requested change is explicit, localized, and does not require design judgment.

Micro Task Mode applies only when all of these are true:

- the requested outcome is unambiguous
- the change is localized to a small, known area
- there is no meaningful product, architecture, security, data, migration, or API contract decision
- no new behavior needs to be designed
- no root-cause investigation is needed
- the likely verification path is a simple diff review, syntax check, targeted test, or smoke check

### Quantitative guard

File count alone does not decide Micro Task Mode. Scope and risk decide it.

If any of these is true, the task is not a micro task regardless of how mechanically simple the edit looks:

- the change spans multiple subsystems
- the change modifies public behavior, access control, persistence, schema, migrations, external API contracts, or release/deploy flow
- the request requires debugging, root-cause analysis, design choice, or product interpretation
- a new or updated test is needed to define or prove the intended behavior
- verification cannot be reduced to a small, relevant check

### Anti-rationalization patterns

These are not valid arguments for Micro Task Mode:

- "mechanical / search-and-replace change" — repeated edits can still create broad miss risk
- "display-only / formatting change" — if tests, layout, translations, or downstream consumers depend on it, risk is nonzero
- "small code diff" — a small diff can still affect auth, data, contracts, routing, or global invariants
- "the user said it is simple" — use the user's scope signal, but verify the actual risk surface

### Micro Task execution

In Micro Task Mode:

- skip spec, plan, TDD, subagent evaluation, stage transition markers, and stage reference loads
- make the smallest direct change that satisfies the request
- review the diff for unintended edits
- run only the smallest relevant verification
- report what changed and how it was checked

Micro Task Mode must still respect:

- local guardrails (AGENTS.md, CLAUDE.md)
- scope discipline
- destructive-action confirmation requirements

If new ambiguity, risk, behavior design, or a larger verification surface appears while executing a micro task, stop using Micro Task Mode and reclassify before continuing.

When in doubt, classify by the actual risk surface, not by edit size.

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
- [references/agent-recovery.md](references/agent-recovery.md): agent loop, drift, and degradation detection and recovery

The main file defines the contract. The reference files define how to execute each stage.

## Context Management

### Context budget tiers

Be aware of how much context has been consumed during the session. Adjust behavior as context fills:

| Tier | Utilization | Behavior |
|------|------------|----------|
| PEAK | 0–30% | Full exploration, detailed traces, read broadly |
| GOOD | 30–50% | Normal execution, read what is needed |
| DEGRADING | 50–70% | Prefer targeted reads over broad exploration, keep review traces focused on changed code, avoid re-reading unchanged files |
| POOR | 70%+ | Finish current task minimally, suggest compaction before starting new work, do not start new exploration |

### Degradation warning signs

Watch for these signals that context quality is dropping:

- review traces becoming shallow or generic
- skipping checklist items or adversarial questions
- making claims without running verification
- losing track of prior findings or plan progress
- responses becoming vague where they were previously specific

If 2+ of these appear, enter agent recovery — load [references/agent-recovery.md](references/agent-recovery.md).

### Context-rot countermeasures

When work is large or long-running:

- prefer bounded fresh subagents over carrying every detail in the main context
- summarize decisions at stage boundaries so the session can recover after compaction
- keep external research notes short and decision-oriented instead of pasting long references into the main thread
- re-open source material when needed rather than trusting degraded memory

### Strategic compaction

Do not wait for automatic compaction to disrupt the workflow. Suggest manual compaction at natural stage boundaries:

**Good compaction points:**

- after spec is written, before planning starts
- after plan is written, before implementation starts
- after implementation reaches GREEN, before review starts
- after review loop completes, before verification starts

**Bad compaction points:**

- mid-implementation between TDD cycles
- mid-review between finding and patching
- while waiting for a subagent to return

When suggesting compaction, include a brief state summary so context can be recovered:

```
현재 상태: [stage] 단계, [task X/N] 진행 중
완료: [what is done]
남은 작업: [what remains]
현재 블로커: [if any]
```

## User Signal Detection

When the user expresses frustration or redirects the approach, treat it as an immediate process reset signal. Do not continue the current path.

**Reset signals** — any of these means "stop and re-investigate from scratch":

- "Stop guessing" / "추측 그만"
- "That's not it" / "그게 아니라"
- "You're going in circles" / "계속 반복하고 있잖아"
- "Think harder" / "더 생각해봐"
- "Read it again" / "다시 읽어봐"
- Repeating the same instruction a second time
- Rejecting two consecutive proposed approaches

**On reset signal:**

1. stop the current action immediately
2. re-read the original error, spec, or user request from source — do not rely on your summary of it
3. re-enter the appropriate stage from the beginning (debugging → step 1, implementation → re-read plan, review → re-capture diff)
4. if the same approach was already tried, form a materially different hypothesis before proceeding

Do not apologize and retry the same thing. The signal means the current approach is wrong, not that it needs one more attempt.

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
