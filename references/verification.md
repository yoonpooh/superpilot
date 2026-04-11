# Verification Stage

## Purpose

Do not claim the work is complete until the relevant verification has been run and checked.

Evidence before claims. Always.

## Verification Flow

For every completion claim:

1. identify the command or check that proves it
2. run it now
3. read the output and exit status
4. confirm whether the result actually supports the claim
5. only then report success

Skip any step and the claim is unverified.

## Required Mapping

Match verification strength to the change:

- local logic change -> focused tests
- contract, boundary, or persistence change -> integration tests
- user workflow change -> E2E or strongest available alternative
- config, build, or CI change -> build plus smoke checks
- skill or workflow-rule change -> loadability, consistency, and targeted text validation

If ideal coverage is unavailable, run the strongest available alternative and report the gap briefly.

## Common Claim Mapping

- “tests pass” -> run the actual test command and confirm zero failures
- “build succeeds” -> run the build command and confirm exit 0
- “bug is fixed” -> run the original repro or proving test and confirm the symptom no longer occurs
- “regression is covered” -> show the failing-then-passing test path or equivalent evidence
- “task is complete” -> verify tests/build/manual checks and re-check the plan/spec coverage

## Not Enough

These are not sufficient:

- “it should work now”
- a previous test run from earlier in the session
- partial checks when the claim is broader
- subagent success reports without independent confirmation
- linter success as a substitute for a build or runtime check
- a focused test when the claim is about the whole feature

## Completion Claims

Before saying any of these, run fresh verification:

- tests pass
- build succeeds
- bug is fixed
- regression is covered
- task is complete

## Red Flags

Stop if you are about to say any version of:

- “should work now”
- “probably fixed”
- “looks correct”
- “the tests should pass”
- “the subagent already checked it”

Those are confidence statements, not verification.

## Coverage Check

Before final completion, re-read the current spec and plan and confirm:

- every in-scope requirement is implemented
- every critical behavior has a proving check
- any remaining gap is explicitly called out and is genuinely non-blocking

## Final Summary

The completion summary should cover:

- what changed
- why it changed
- what was verified
- any remaining non-blocking gap

Do not report success first and evidence second. Evidence must come first.

## Delegation Rule

Subagents may report their own checks, but the main agent must still verify the integrated result independently before making completion claims.

## Minimum Final Check

Before the final handoff:

1. re-check the current spec
2. re-check the plan
3. run the strongest relevant verification
4. confirm the diff matches the claimed result
5. only then write the completion summary
