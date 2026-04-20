# Implementation Stage

## Purpose

Execute the current plan with tight scope control, strong TDD discipline, and minimal waste.

Autonomy matters here: the goal is to keep moving from plan to verified implementation without routine user intervention, while still stopping for real safety or ambiguity gates.

## Execution Setup Gate

Before editing production code, confirm the execution environment is appropriate for autonomous work:

1. check whether the current tree is clean enough for in-place work
2. if not, decide whether isolated branch or worktree execution is needed
3. identify the proving command(s) that define a clean baseline for this task
4. if the task is large or parallelizable, decide which slices should move to fresh subagents

For risky work, do not drift into implementation without an explicit workspace and baseline decision. Clean diff capture is part of quality, not a convenience.

## Pre-Edit Investigation Gate

Before writing or editing any production file, confirm that you have read and understood:

1. the file being changed — read it, do not rely on memory or assumptions
2. all direct importers/callers of the changed symbol — grep for usages
3. related schema, config, or type definitions that constrain the change
4. the current spec and plan entries for this change

For bootstrap, middleware, config, or request-entry changes, caller tracing alone is not enough. Also investigate execution order:

- what runs immediately before the changed code in the real request lifecycle
- what runs immediately after it
- which existing global security, canonicalization, validation, or auth steps must happen before sensitive input is processed
- whether the new logic bypasses or precedes any of those steps

If the change touches redirect, auth, middleware, canonicalization, token, cookie, query param, or session handling, explicitly confirm the new code does not consume or act on sensitive input before the existing global invariants unless the spec intentionally requires that behavior.

If any of these have not been done for the current edit target, do the investigation first. Do not edit a file you have not read in this session.

This gate applies to every edit, including "obvious" one-line fixes. The cost of reading first is low; the cost of editing with wrong assumptions is a broken review loop.

Exception: test files being created for the first time do not require caller investigation (they have no callers yet).

## Research-First Execution

If execution depends on framework behavior, third-party APIs, standards, or toolchain facts that are not stable from repo context alone:

- pause and research the fact before editing
- prefer official docs, specs, or primary sources
- bring back only the decision-relevant conclusion

Do not guess unfamiliar library behavior and “see if tests fail” when the correct answer is cheaply knowable from source material.

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
- test runner or dependencies not installed
- vendor directory missing

Test code itself proves intent. Writing the test is mandatory even when the execution environment is unavailable. Record that verification was deferred, but do not skip the test.

### Existing test as TDD trigger

If an existing test would break from your change, that change **has a test surface by definition**. The existing test proves it. TDD applies unconditionally:

1. change the test assertion to the new expected value → RED
2. confirm it fails for the expected reason (old behavior still in production code)
3. make the production change → GREEN
4. proceed to review

"The change is just a format string / config value / display tweak" is not an exception when a test already asserts the old value. The test's existence is the signal, not your judgment of the change's complexity.

### Valid narrow exceptions

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

## Requirement Ledger Discipline

Before starting each slice:

- identify which requirement(s) from the spec ledger this slice satisfies
- confirm the planned test or proving check covers those requirements

After finishing each slice:

- confirm the implemented behavior still matches the requirement wording
- confirm no requested outcome was quietly deferred or dropped

If a requirement no longer makes sense, stop and return to spec/plan clarification instead of silently narrowing the scope in code.

## Bug Fix Discipline

For bug fixes:

- trace the full reproduction path before patching
- map where relevant state is created, transformed, stored, restored, and rendered
- avoid fixing only the first visible symptom if multiple consumers exist
- use [debugging.md](debugging.md) when the root cause is not already proven

## Drift-Sensitive Changes

When the task changes any of these, treat all paired artifacts as part of the same slice:

- ORM or database schema
- API request/response contract
- generated clients or type definitions
- fixtures, seed data, snapshots, or docs that define the contract externally

If the code change requires a migration, schema update, codegen refresh, fixture update, or doc update to stay truthful, that paired change is not optional cleanup — it is part of the implementation.

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
- “edit this line to X” without TDD context

When subagents are used for implementation, prefer fresh context per slice. Reusing one subagent across unrelated tasks increases context rot and hidden carry-over assumptions.

### Subagent TDD Ownership

The main agent owns the TDD process. Delegating implementation does not delegate TDD responsibility.

When dispatching a subagent for implementation work:

- Option A: main agent writes the failing test first, confirms RED, then dispatches subagent with “make this test pass” as the goal
- Option B: subagent prompt explicitly includes the full TDD sequence — “write failing test first, confirm RED, then implement”

Never dispatch a subagent with edit-only instructions when the change has a test surface. A prompt like “change line N to X” skips TDD by construction.

After subagent returns:

- verify that a test was written if the change has a test surface
- verify that the test actually tests the changed behavior, not just exists
- run the test yourself — do not trust “tests pass” in the subagent report

## Test Failure Triage

When a test fails during implementation, classify it before spending time fixing it.

### Classification

| Category | Definition | Action |
|----------|-----------|--------|
| **In-branch** | Failure caused by changes in the current task | Fix immediately — this is your responsibility |
| **Pre-existing** | Failure exists on the base branch before your changes | Do not fix unless it is in scope. Verify by checking out the base branch or using `git stash` to confirm the failure exists without your changes |
| **Flaky** | Failure is non-deterministic and unrelated to your changes | Re-run once to confirm flakiness. If flaky, note it and move on. Do not spend time debugging intermittent failures outside your scope |

### Triage procedure

1. test fails
2. check: does this test touch code you changed? (grep the test for your changed symbols)
3. if yes → likely **in-branch**, investigate and fix
4. if no → run `git stash && [test command] && git stash pop` or check base branch
5. if the failure reproduces without your changes → **pre-existing**, skip
6. if the failure does not reproduce consistently → **flaky**, note and skip

### Do not

- spend 30 minutes debugging a test that was already failing before your changes
- silently skip a failing test without classifying it
- disable or delete a pre-existing failing test to make your changes "green"
- treat a flaky test as a blocker when it is not related to your changes

### Reporting

When skipping a pre-existing or flaky failure, note it briefly in the completion summary:

```
참고: `test_user_session_timeout` — 기존 실패 (base branch에서도 실패 확인)
```

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
- do not treat missing migration/codegen/doc/fixture updates as “follow-up” when the changed behavior depends on them

### Bundled Changes and TDD

When multiple changes target the same file, evaluate each change independently for test surface.

- do not classify all changes as “non-testable” because some are CSS-only or config-only
- if one change in the bundle has a test surface, that change requires TDD regardless of what the other changes are
- when bundling testable and non-testable changes, execute the testable ones through TDD first

A file boundary is not a TDD boundary. Behavior boundaries are.

## Red Flags

Stop and correct course if you catch yourself thinking:

- “I will write tests after the code”
- “this is too simple to test”
- “let me just fix it quickly without reproducing”
- “the subagent said it is done, that is probably enough”
- “tests are green, so the task is done — I can skip review”
- “while I am here, I should clean up these unrelated files too”
- “all changes are in one file, so they are all the same type”
- “test runner is not available, so I will skip writing the test”
- “I told the subagent to edit this line — no TDD needed for a simple edit”
- “I'm following the spirit of TDD even though I wrote the code first”
- “this is just a refactor, the behavior isn't changing”
- “I already tested this manually, a written test is redundant”
- “the existing tests are weak, so adding one more won't help”
- “let me just get it working first, then I'll add tests”
- “this is an MVP / simple version / basic structure — tests come later”
- “format/display change, not logic — TDD unnecessary”
- “same pattern across N files — effectively single-line, so trivial”
- “mechanically simple, so the process overhead exceeds the risk”
- “existing test will break but it's just updating an assertion, not real TDD”

These are rationalization patterns, not valid exceptions. The rules exist because these exact shortcuts have caused failures before. “Following the spirit” while violating the letter is still a violation — do not use the intent behind a rule to justify breaking it.

## Stage Exit Gate

GREEN does not mean done. GREEN means the implementation is ready for review.

After all implementation slices are complete and green:

1. stop writing production code
2. proceed to the Review stage — load [review.md](review.md) and run the full review loop
3. do not write a completion summary, do not claim the task is done, do not move to verification

No path from this stage leads to completion without passing through the Review stage first. If you find yourself about to report results to the user after GREEN without having loaded and executed review.md, you are skipping a mandatory stage.

Exit marker: `## IMPLEMENTATION GREEN`
