# Implementation Stage

## Purpose

Execute the current plan with tight scope control, strong TDD discipline, and minimal waste.

## TDD Rules

Core principle:

```text
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

For feature work, bug fixes, refactoring, and behavior changes:

- write the test first when an executable test surface exists
- run the test and confirm the expected failing state
- implement the smallest correct change to pass
- rerun the focused test and any relevant surrounding checks
- refactor only after returning to green

If you wrote production code first for a behavior change with a valid test surface, do not preserve that shortcut as the “real” implementation. Rewrite the flow so the test proves the behavior first.

Do not use these as excuses to skip TDD:

- MVP
- simple version
- basic structure first
- existing project has weak tests

Valid narrow exceptions:

- copy-only change
- config-only change with no executable surface
- workflow text change with no runnable behavior

If skipping TDD, state the exact reason in the plan or execution notes.

## Red-Green-Refactor

For each meaningful behavior slice:

1. RED: write a failing test for one behavior
2. VERIFY RED: confirm it fails for the expected reason
3. GREEN: write the smallest change that passes
4. VERIFY GREEN: rerun the focused test and relevant surrounding checks
5. REFACTOR: clean up only while staying green

Do not bundle multiple behaviors into one test-first cycle if they can fail independently.

### Verify RED

Confirm all of these:

- the test fails, not errors unexpectedly
- it fails for the expected reason
- it proves missing or broken behavior, not a typo in the test

If the test passes immediately, it is not proving the new behavior yet.

### Verify GREEN

Confirm all of these:

- the focused test passes
- relevant surrounding tests still pass
- no obvious warnings or side-effect failures appeared

If surrounding checks fail, fix that before calling the slice complete.

## Scope Discipline

- change only what serves the current spec
- do not perform unrelated cleanup
- do not opportunistically refactor neighboring code unless it directly supports the requested work
- preserve existing patterns unless the spec intentionally changes them

## Bug Fix Discipline

For bug fixes:

- trace the full reproduction path before patching
- map where relevant state is created, transformed, stored, restored, and rendered
- avoid fixing only the first visible symptom if multiple consumers exist
- use [debugging.md](debugging.md) when the root cause is not already proven

## Execution Strategy

### Direct execution

Use direct execution for tightly coupled or sequential work. Keep momentum and avoid coordination overhead.

### Subagent execution

Use subagents only when independence is real.

Good subagent tasks:

- implement a bounded module change
- investigate a bounded code path
- add tests in a disjoint test file
- perform a bounded first-pass review of an isolated file group

Bad subagent tasks:

- “look around and tell me what you think”
- editing overlapping files
- splitting one tightly coupled function chain into multiple owners

## Blocker Handling

When blocked by a technical issue:

1. try up to 3 materially different approaches when autonomous resolution is realistic
2. report immediately if the blocker is clearly environmental, credential-related, or permission-related
3. after 3 failed approaches, summarize:
   - what was tried
   - the exact blocker
   - the best next step

Do not silently stop at the first failure.

## Execution Integrity

- do not trust subagent success blindly
- review and integrate their work yourself
- do not let “looks correct” replace running tests
- do not widen scope while implementing just because adjacent code is tempting to clean up

## Red Flags

Stop and correct course if you catch yourself thinking:

- “I will write tests after the code”
- “this is too simple to test”
- “let me just fix it quickly without reproducing”
- “the subagent said it is done, that is probably enough”
- “tests are green, so the task is done — I can skip review”
- “while I am here, I should clean up these unrelated files too”

These are shortcut patterns that reduce trust.

## Stage Exit Gate

GREEN does not mean done. GREEN means the implementation is ready for review.

After all implementation slices are complete and green:

1. stop writing production code
2. proceed to the Review stage — load [review.md](review.md) and run the full review loop
3. do not write a completion summary, do not claim the task is done, do not move to verification

No path from this stage leads to completion without passing through the Review stage first. If you find yourself about to report results to the user after GREEN without having loaded and executed review.md, you are skipping a mandatory stage.
