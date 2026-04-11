# Plan Stage

## Purpose

Turn the current written spec into an executable implementation plan that another strong engineer could follow without guessing.

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
- do not ask the user to choose the execution mode
- do not put commit or PR steps into the plan unless the user explicitly asked for publishing workflow
- do not leave placeholders, vague follow-ups, or “figure it out later” instructions

## Plan Structure

Use this shape:

```markdown
# <Task Name> Implementation Plan

**Goal:** ...

**Architecture:** ...

**Execution Mode:** direct / subagent / mixed

**Verification:** ...

---

### Task 1: ...

**Files:**
- Modify: `...`
- Test: `...`

- [ ] Step 1: ...
- [ ] Step 2: ...

### Task 2: ...
```

Every task should answer:

- what files are touched
- what behavior changes
- what test proves the change
- what verification proves readiness

## Task Granularity

Prefer steps that are small enough to execute and verify cleanly:

- write the failing test
- run it and confirm the intended failure
- implement the minimal code
- rerun focused tests
- run broader relevant checks

Do not collapse an entire subsystem change into one vague task if it can be decomposed safely.

## Decomposition Rules

Prefer small, coherent tasks.

Split by:

- bounded file groups
- independent behavior slices
- testable units

Do not split by arbitrary technical layers if that increases coordination cost.

## Direct vs Subagent Execution

Decide execution mode yourself.

### Use direct execution when:

- the work is sequential
- outputs of one step feed the next
- the same files will be edited repeatedly
- the reasoning is tightly coupled

### Use subagents when:

- tasks are independent
- write targets do not overlap
- module boundaries are clear
- parallel work saves real time

### Subagent constraints

- assign one self-contained responsibility per subagent
- give each subagent an explicit file scope
- do not split a single tightly coupled call chain across multiple subagents
- treat subagents as accelerators, not final owners
- do not assign overlapping write targets to multiple subagents
- ask the subagent to report focused verification results, but do not treat that as final verification

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

## Plan Self-Review

Before execution starts, review the plan for:

- scope coverage
- placeholder text
- task ordering errors
- overlapping ownership between planned subagents
- missing verification steps

The main agent remains responsible for:

- final integration
- final review
- final verification
