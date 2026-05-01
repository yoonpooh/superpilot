# Plan Stage

## Purpose

Turn the current written spec into an executable implementation plan that another strong engineer could follow without guessing.

Skip this stage when the request qualifies for Micro Task Mode as defined in `SKILL.md`.

The plan is mandatory for non-trivial work.

## Output Path

Write the plan to:

```text
~/.superpilot/docs/<repo-name>/plans/YYYY-MM-DD-<short-slug>.md
```

## Planning Rules

- plan after the spec is written and clarified enough to execute, not before
- decompose by responsibility and file boundaries
- keep steps concrete and executable
- include tests and verification in the plan
- preserve the full requirement ledger from the spec; do not let planning silently narrow the task
- do not ask the user to choose the execution mode
- do not put commit or PR steps into the plan unless the user explicitly asked for publishing workflow
- do not leave placeholders, vague follow-ups, or “figure it out later” instructions

## Plan Structure

Use this shape:

```markdown
# <Task Name> Implementation Plan

**Goal:** ...

**Requirement Coverage:** ...

**Architecture:** ...

**Execution Mode:** direct / subagent / mixed

**Workspace Strategy:** in-place / isolated branch / isolated worktree

**Verification:** ...

---

### Task 1: ...

**Files:**
- Modify: `...`
- Test: `...`

**Ownership:** main agent / subagent

**Requirements Covered:** ...

- [ ] Step 1: ...
- [ ] Step 2: ...

### Task 2: ...
```

Every task should answer:

- what files are touched
- what behavior changes
- what test proves the change
- what verification proves readiness
- which requirement(s) from the spec ledger it covers
- who owns the slice and whether a fresh subagent context is needed

## Task Granularity

Prefer steps that are small enough to execute and verify cleanly:

- write the failing test
- run it and confirm the intended failure
- implement the minimal code
- rerun focused tests
- run broader relevant checks

Do not collapse an entire subsystem change into one vague task if it can be decomposed safely.

As a default heuristic, prefer slices that a strong engineer or fresh subagent could complete without guessing in roughly 2-15 minutes of focused work. The goal is not speed theater — it is low ambiguity and clean verification.

## Decomposition Rules

Prefer small, coherent tasks.

Split by:

- bounded file groups
- independent behavior slices
- testable units

Do not split by arbitrary technical layers if that increases coordination cost.

## Workspace Strategy

Choose the execution workspace explicitly in the plan.

### Use in-place execution when:

- the tree is clean or unrelated changes are easy to isolate mentally
- the task is bounded and rollback risk is low
- the same files will be touched iteratively by the main agent

### Use an isolated branch or worktree when:

- the working tree is already dirty
- multiple parallel implementation or review threads are likely
- the task is risky enough that clean diff capture matters
- you want independent verification or subagent work without cross-contamination

Do not ask the user to choose unless workspace setup itself would be destructive or conflicts with an explicit repository policy.

## Direct vs Subagent Execution

Decide execution mode yourself.

### Delegation evaluation gate

For every non-trivial plan, evaluate subagent use before finalizing task ownership. This is mandatory even when the final choice is direct execution.

The plan must include:

- the chosen mode: `direct`, `subagent`, or `mixed`
- candidate slices that could run in fresh subagents
- the ownership and file scope for each chosen subagent slice
- what the main agent will do locally while subagents run
- a short reason if all work remains direct

Prefer `mixed` execution when at least one independent slice can run safely while the main agent continues non-overlapping work.

### Use direct execution when:

- the work is sequential
- outputs of one step feed the next
- the same files will be edited repeatedly
- the reasoning is tightly coupled
- delegation overhead would exceed the parallelism benefit, and the plan records why

### Use subagents when:

- tasks are independent
- write targets do not overlap
- module boundaries are clear
- parallel work saves real time
- there are independent investigation questions that can be answered in parallel
- tests or fixtures can be added in disjoint files while production work continues elsewhere
- a bounded first-pass review can run before the main final review

### Subagent constraints

- assign one self-contained responsibility per subagent
- prefer a fresh subagent per independent task so stale context does not leak between slices
- give each subagent an explicit file scope
- do not split a single tightly coupled call chain across multiple subagents
- treat subagents as accelerators, not final owners
- do not assign overlapping write targets to multiple subagents
- ask the subagent to report focused verification results, but do not treat that as final verification
- every subagent prompt for implementation work must include TDD instructions when the change has a test surface
- never write subagent prompts as bare edit instructions ("change line N to X") — include the TDD sequence or have the main agent write the failing test first

### Subagent model selection

When the runtime supports model selection for subagents, match model capability to task complexity:

| Task type | Model tier | Examples |
|-----------|-----------|----------|
| Mechanical / repetitive | Lighter model (e.g., haiku, fast mode) | Rename symbol across files, add boilerplate tests, format conversions |
| Standard implementation | Default model | Implement a bounded feature slice, write integration tests, fix a well-understood bug |
| Architecture / judgment | Strongest available model | Design a new abstraction, resolve conflicting patterns, review complex state management |

Do not default every subagent to the strongest model. Lighter models are faster and consume less context budget for tasks that do not require deep reasoning. When unsure, use the default model.

## Subagent Review Loop

When subagents are used, keep the quality loop explicit:

1. dispatch the subagent with a bounded scope
2. require the subagent to report:
   - what changed
   - what verification it ran
   - any concerns or open questions
3. review the returned changes for spec compliance first
4. review the returned changes for code quality second
5. only then merge the slice into the integrated work

Do not let a subagent's self-confidence replace review.

## Plan Challenge Passes

Before execution, pressure-test the plan through the lenses relevant to the task. Use only the lenses that matter.

- Product / scope: is the plan solving the real request, or only the implementation-shaped surface of it?
- Engineering: are architecture, error paths, state flow, and tests concrete enough to execute without guessing?
- Design / UX: for user-facing work, will the result feel intentional rather than merely functional?
- DevEx / onboarding: for tooling, docs, or setup flows, does the plan preserve a short path to first success?
- Security: for auth, secrets, permissions, or external execution, is the threat model and verification path explicit?

Fold challenge findings back into the plan before implementation starts.

## Scope Reduction Detection

Before execution, compare the plan against:

- the original user request
- the spec's requirement ledger
- the chosen design summary

If any requested or implied requirement has no owning task and no stated non-goal, the plan is incomplete. Add the missing task or explicitly narrow scope with a reason.

## Plan Self-Review

Before execution starts, review the plan for:

- scope coverage
- placeholder text
- task ordering errors
- overlapping ownership between planned subagents
- missing verification steps
- silent scope reduction relative to the user request or requirement ledger
- weak workspace strategy for the actual task risk

The main agent remains responsible for:

- final integration
- final review
- final verification

## Stage Exit

This stage is complete when:

- the plan file is written and self-reviewed
- execution mode is chosen
- task ordering and ownership are clear
- no placeholder text remains

Exit marker: `## PLAN COMPLETE`
