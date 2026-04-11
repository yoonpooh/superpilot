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
- passing tests without a fresh final review pass

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

Before final completion of an implementation task, re-read the current spec and plan and confirm:

- every in-scope requirement is implemented
- every critical behavior has a proving check
- any remaining gap is explicitly called out and is genuinely non-blocking
- the last review pass happened after the last code patch and still found zero actionable findings
- the work was not closed merely because tests turned green again

## Final Summary

The completion summary should be easy to scan in 5-10 seconds.

Use this shape by default:

```md
## 완료 요약
- 검증: 실행한 핵심 검증과 결과
- 리뷰: review loop N회, findings X건 패치, 최종 0건
- 변경: 무엇을 바꿨는지 한 줄
- 이유: 왜 바꿨는지 한 줄
- 범위: 핵심 파일/영향 범위
- 참고: 남은 비차단 갭이 있을 때만
```

The completion summary should cover:

- what was verified
- how the review loop ended
- what changed
- why it changed
- any remaining non-blocking gap

Do not report success first and evidence second. Evidence must come first.

Prefer concrete review stats when available:

- number of review loops
- number of findings patched
- final actionable findings count
- whether a fresh final review pass was run after the last patch

## Delegation Rule

Subagents may report their own checks, but the main agent must still verify the integrated result independently before making completion claims.

## Minimum Final Check

Before the final handoff for an implementation task:

1. re-check the current spec
2. re-check the plan
3. confirm the review loop reached zero findings
4. confirm a fresh final review pass after the last patch also reached zero findings
5. run the strongest relevant verification
6. confirm the diff matches the claimed result
7. only then write the completion summary

If step 4 is missing, do not claim the task is complete even if tests or builds are green.

## Review-Only Final Check

Before the final handoff for a review-only task:

1. confirm the task stayed in review-only mode
2. confirm no code was patched during the review
3. confirm the findings are backed by the reviewed artifact
4. confirm the findings are ordered by severity
5. confirm residual risks or testing gaps are noted when relevant
6. only then deliver the findings report
