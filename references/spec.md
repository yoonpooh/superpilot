# Spec Stage

## Purpose

Use this stage to turn an initial request into a clear design and written spec before planning or implementation starts.

Skip this stage only when the task is truly trivial under the exception defined in `SKILL.md`.

For non-trivial work:

- explore context first
- collaborate with the user on the design
- write the spec
- continue into planning once the request is clear enough to describe correctly

Do not implement before the spec/design is clear enough to write down concretely.

## Context Pass

Before asking detailed questions:

- resolve the repository root
- read repo-root agent instruction files such as `AGENTS.md` and `CLAUDE.md` if present
- inspect the relevant files, docs, and recent changes
- identify whether the request is narrowly scoped or crosses multiple subsystems

If the request actually contains multiple independent projects, decompose it before writing a single spec.

## Conversation Discipline

- ask one focused question at a time when possible
- prefer concrete questions over broad brainstorming prompts
- summarize understanding before proposing a design
- if there are real alternatives, propose 2 or 3 approaches with trade-offs
- recommend one approach and explain why

## Clarification Rules

Ask clarifying questions only when they prevent building the wrong thing.

### Assumptions-first approach

Instead of asking open-ended questions, analyze the codebase first and present structured assumptions for the user to correct:

1. read the relevant code, tests, config, and recent git history
2. form assumptions about scope, behavior, constraints, and approach
3. present assumptions with confidence levels:
   - **Confident** — evidence from code makes this clear (e.g., "DB uses PostgreSQL based on schema.prisma")
   - **Likely** — reasonable inference from patterns (e.g., "auth follows the same middleware pattern as existing routes")
   - **Unclear** — multiple valid directions, need user input (e.g., "should deleted items be soft-deleted or hard-deleted?")
4. the user corrects only what is wrong — no need to confirm what is obviously right

This reduces question count and shows the user you already understand their codebase.

### When to ask directly instead

Use direct questions when:

- no existing code provides evidence for the assumption
- the decision is purely about product intent, not technical implementation
- conflicting patterns exist in the codebase and both are reasonable

### Good clarifications

- missing behavior definition
- unclear scope boundaries
- conflicting requirements
- undefined success criteria

### Bad questions

- asking whether to proceed
- asking whether to write a spec or plan
- asking whether to use TDD
- asking whether to use subagents
- asking something the code already answers

Ask one focused question at a time when possible.

## Design Conversation

Before writing the spec:

- summarize your understanding
- propose 2 or 3 approaches when there is a real design choice
- recommend one approach and explain why
- present the design in sections scaled to the task complexity

Cover the parts that matter for implementation:

- architecture or change shape
- touched components or modules
- data flow or control flow
- edge cases and failure handling
- testing approach

Do not ask for approval once the design is clear enough to write and execute safely. The original task request is the authorization to continue, unless a safety gate is triggered.

## Spec File

Write the spec to:

```text
~/.superpilot/docs/<repo-name>/specs/YYYY-MM-DD-<short-slug>.md
```

Use this structure:

```markdown
# <Task Name> Spec

## Goal

## Context

## Scope

## Proposed Design

## Execution Strategy

## Edge Cases and Risks

## Testing Strategy

## Execution Readiness
```

Keep the spec concrete. Avoid placeholders and vague statements like:

- placeholder markers
- add proper validation
- handle edge cases later
- improve architecture as needed

## Spec Self-Review

Before transitioning to planning:

1. scan for placeholders or vague wording
2. check internal consistency across sections
3. confirm the design matches the requested scope
4. confirm the testing strategy actually covers the proposed change

## Execution Transition

After the spec is written:

- present the spec path
- summarize the selected design briefly
- state that execution will continue from this spec

Then:

- move to the plan stage
- do not ask for approval at routine execution checkpoints

## Stage Exit

This stage is complete when:

- the spec file is written and self-reviewed
- the spec path has been presented
- the selected design has been summarized

Exit marker: `## SPEC COMPLETE`
