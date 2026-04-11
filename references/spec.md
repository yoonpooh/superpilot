# Spec Stage

## Purpose

Use this stage to turn an initial request into a clear design and written spec before planning or implementation starts.

For non-trivial work:

- explore context first
- collaborate with the user on the design
- write the spec
- continue into planning once the request is clear enough to describe correctly

Do not implement before the spec/design is clear enough to write down concretely.

## Context Pass

Before asking detailed questions:

- resolve the repository root
- read repo-root `AGENTS.md` if present
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

Good clarifications:

- missing behavior definition
- unclear scope boundaries
- conflicting requirements
- undefined success criteria

Bad questions:

- asking whether to proceed
- asking whether to write a spec or plan
- asking whether to use TDD
- asking whether to use subagents

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
