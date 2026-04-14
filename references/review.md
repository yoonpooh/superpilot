# Review Stage

## Purpose

Review should be stricter than implementation. The goal is to prove the change is ready, not to feel good about it.

For implementation tasks, review the diff until actionable findings reach zero.

By default, review is an internal quality gate, not a user-facing report.

If the task itself is review-only, this stage becomes the primary deliverable:

- inspect the diff with the same harsh standard
- return findings ordered by severity
- do not patch unless the user explicitly asks for fixes
- capture the review artifact from its actual source (local diff, branch diff, commit diff, PR patch, changed files)

## Review Scope

- capture the actual changeset from the active source, not only plain `git diff`
- review tracked changes, staged changes, and any untracked files that are part of the task
- plain `git diff` alone is not sufficient when the work includes new untracked files
- if a new file is part of the work, either read it directly during review or stage it before the final whole-diff review pass
- review the changed lines first, then read as far beyond the immediate calling context as needed to complete the mandatory traces and contract checks correctly
- do not pad the review with unrelated unchanged code

## Mandatory Trace Activities

These are not checklist items to glance at. They are concrete activities that require reading and following actual code paths in the diff. Do all that apply.

### 1. Failure path trace

For every external call, network request, or operation that can fail in the diff:

- identify every failure branch (exception, error response, timeout, empty result)
- trace each failure from the point of origin through to what the user sees
- verify that user-visible state, session state, and persisted state all stay consistent on failure
- if the diff creates any user-visible artifact before success is confirmed, verify there is an explicit failure-state or correction path
- if the failure is swallowed or only partially handled, that is a finding

### 2. State consistency trace

For every place the diff updates state in two or more locations (e.g., DOM and server, client array and session, cache and database):

- trace what happens when the operation succeeds
- trace what happens when the operation fails partway through
- verify that the two states cannot diverge
- if the diff uses an optimistic state transition, verify rollback, reconciliation, or explicit failed state exists
- if repeated retries or duplicate triggers could create duplicate user-visible results or duplicate persisted state, that is a finding
- if they can diverge, that is a finding

### 3. Access and cost trace

For every new route, endpoint, or entry point in the diff:

- compare its access control (auth, permissions, roles) against the existing project pattern
- if it calls a paid external API or resource-heavy operation, verify that rate limiting or throttle exists
- if the new endpoint has weaker protection than similar existing endpoints, that is a finding

### 4. Input boundary trace

For every user input that reaches the diff:

- trace what happens with empty, whitespace-only, maximum-length, and malformed input
- verify that validation catches these before they reach expensive operations
- if invalid input can trigger an external API call or corrupt state, that is a finding

### 5. Entry point coverage trace

When the diff modifies behavior reachable through a specific entry point:

- identify all entry points in the codebase that trigger the same behavior or reach the same code path
- verify that the change applies correctly to every entry point, not just the one you happened to test
- if an alternate entry point bypasses the changed code (e.g., a chatbot widget calls an API directly instead of going through the shared cart function), that is a finding
- do not assume "I fixed function X, so all callers of X are covered" — verify that all callers actually go through X

This trace catches the pattern where one entry point is fixed but a parallel entry point that does the same thing through a different code path is missed.

If a trace activity does not apply to the current diff (e.g., no external calls, no new endpoints), skip it and state why it does not apply.

## Review Loop

For implementation tasks, repeat this loop:

1. capture the current diff
2. read every changed hunk
3. run mandatory trace activities (see below) — these are concrete code-path traces, not checkbox items
4. walk through every checklist item against every changed block — record what you checked and what you found per item
5. answer every adversarial question — record each answer
6. collect actionable findings
7. if findings > 0: patch them, then go back to step 1 — **do not exit here**
8. if findings = 0: this pass is a candidate for exit — check the exit condition below

### Findings carry-over on revert and reimplementation

When a revert-and-reimplement cycle occurs (e.g., TDD restart, approach change):

- all findings from prior review passes remain active constraints
- before exiting the review loop on the reimplemented code, verify that every prior finding is either resolved in the new implementation or explicitly marked as no longer applicable with a reason
- "the code was reverted" does not erase a finding — the finding describes a behavior gap that the new implementation must also address
- treat prior findings as a checklist that the reimplementation must satisfy

**The exit point is step 8, never step 7.** Patching a finding changes the diff. A changed diff is an unreviewed diff. You must return to step 1 and review the new diff that includes your patches. The patch itself can introduce new problems (e.g., a try-catch that swallows an exception needed for transaction rollback). Only when a full review pass on the post-patch diff produces zero findings may you exit.

Steps 3, 4, and 5 are not optional. A review pass that skips any of these is not a valid review pass and does not count toward the zero-findings exit condition.

For review-only tasks, run the same trace, checklist, and adversarial process, but stop after producing the findings report. Do not patch as part of the loop unless the user explicitly switches the task to fixing mode.

For implementation tasks, exit only when actionable findings are zero, unless a true safety gate or external blocker appears.

Completion is stricter than "I patched the last finding":

- after the final patch, run at least one fresh whole-diff review pass with no new code changes in between
- the fresh pass must re-capture the diff, re-run traces, walk through every checklist item, and answer every adversarial question against the post-patch diff
- only if that fresh pass also returns zero actionable findings may the task move to final verification and completion
- if a later "code review" request on the same finished diff would obviously find an issue, the task was not actually complete
- a green test run or successful repro does not waive this fresh final pass
- if you patched N findings, you owe at least N+1 review passes total (N passes that found issues + 1 clean pass)

If the same finding repeats for 2 consecutive rounds and you cannot resolve the disagreement with evidence, treat it as a judgment conflict and escalate to the user.

### Stall detection

Track the number of actionable findings per review pass. If the count does not decrease for 3 consecutive passes, the review loop is stalling.

When a stall is detected:

1. stop the loop immediately
2. classify each remaining finding as one of:
   - **fixable now** — clear solution exists but was missed in prior patches
   - **design tension** — fixing it requires a design choice the spec does not cover
   - **false positive** — the finding is not actually actionable on re-examination
3. fix all "fixable now" items in one batch, then restart the loop
4. escalate "design tension" items to the user with a concrete description of the trade-off
5. drop "false positive" items with a one-line reason

If stall persists after classification and re-attempt, escalate to the user with:

- the current findings list
- what was tried
- what decision is needed

Do not loop indefinitely. A stalling review loop wastes context and risks quality degradation.

For implementation tasks, treat findings as internal work items:

- patch them immediately when they are in scope and fixable now
- do not stop to ask the user what to do with normal review findings
- do not present a findings list mid-task unless the user explicitly asked for a review-only outcome

For review-only tasks:

- do not patch findings during the review stage
- do not convert the review into implementation on your own
- return the findings as the deliverable

## Required Procedure

For every review pass:

1. Capture the full changeset from the active source:
   - local implementation work: `git diff`, `git diff --cached`, and any untracked task files via `git ls-files --others --exclude-standard`
   - review-only branch/commit/PR work: capture the equivalent patch, file list, or changed-file content from that artifact
2. Read every changed line carefully
3. For each changed block, answer:
   - what are the possible inputs?
   - what edge cases hit this path?
   - what can fail here?
   - does the caller still match the contract?
   - is this consistent with nearby code patterns?
4. Walk through every checklist item below against the diff — do not skip items, do not batch them as "all fine"
5. Answer every adversarial question below — write out each answer, even when the answer is "no issue"
6. Record findings with exact file references from the diff or changed files so they can be patched precisely

If steps 4 or 5 produced no findings at all, re-read the diff one more time with the assumption that you missed something. A non-trivial change that triggers zero checklist findings on the first pass is almost always under-reviewed, not perfect.

If untracked files are part of the task, the review is incomplete until those files have also been read in full or staged so they appear in the final whole-diff pass.

## Checklist

Check each category explicitly against the diff. Do not skip items. Do not collapse multiple items into "all clear."

For each item, identify the relevant changed code and state what you checked:

- input validation — can invalid, empty, or boundary inputs reach this path?
- error and exception handling — what happens on every failure branch? is the failure surfaced, swallowed, or leaked?
- return-value and caller compatibility — do all callers still match the new contract?
- regression risk to existing callers — could an existing consumer break from this change?
- concurrency and ordering assumptions — are there race conditions, duplicate submissions, or ordering dependencies?
- data and state consistency — can local state, remote state, and user-visible state diverge? if optimistic updates exist, is there rollback, reconciliation, or explicit failed state?
- performance or resource blowups — can this path be called in an unbounded loop or without rate limiting?
- security and sensitive-data exposure — are secrets, PII, or internal details leaked in logs, responses, or error messages?
- test coverage for the most plausible failure mode — is there a test for the first thing that would break?
- actual alignment with the current spec — for implementation tasks, does the implementation match what was specified, not just "work"?
- actual alignment with the requested review target — for review-only tasks with no spec, does the finding reflect the stated scope and artifact under review?
- operability and debuggability — can you diagnose a production failure from logs alone?
- partial failure behavior — if this operation half-succeeds, what state is left behind?
- dead code and unused parameters — are there parameters validated but never sent, or code paths that can never execute?

## Adversarial Questions

Before calling the review complete, answer all of these explicitly — write out each answer:

- If this ships as-is, what is most likely to break first?
- What incorrect assumption does this change make about inputs, ordering, state, or third-party behavior?
- If this change updates visible state before durable success, what corrects it on failure or retry?
- What test would fail immediately if the implementation were subtly wrong in the most plausible way?
- If malicious or malformed input reaches this path, what happens?
- What would a strong reviewer challenge in this diff?
- What would a user encounter on the first error or edge-case path?
- Are there other entry points in the codebase that trigger the same behavior but bypass this change?

If any answer produces a credible problem, that is a finding.

Do not answer these questions in your head. Write out each answer as part of the review work. A one-word "nothing" for every question is a red flag that the review was too shallow.

## User Communication Rule

Default behavior for implementation tasks:

- keep review findings inside the execution loop
- patch and re-review instead of surfacing the issue list
- mention review only in the final completion summary, for example by stating how many review loops were completed and that actionable findings reached zero

Expose findings to the user only when one of these is true:

- the user explicitly asked for review-only output
- a true safety gate was triggered
- an external blocker prevents fixing the finding now
- the same judgment conflict repeats and cannot be resolved with evidence

For review-only requests:

- do not convert the task into implementation work on your own
- do not write a spec or plan unless the user later asks for fixes
- present findings first, then brief residual risks or testing gaps

## Review Mindset

Use a deployment-blocker mindset:

- assume the diff is not ready until it proves otherwise
- do not confuse familiarity with correctness
- do not stop after finding only surface issues
- if the review felt easier than the implementation, review again more deeply
- review as if the user will immediately ask for a separate code review after completion
- for implementation work, review until that later code review should not find any new in-scope issue
- do not let "tests pass now" create completion momentum that skips the final review pass

### Trace quality standard

A trace is not "I looked at this and it seems fine." A trace requires:

- citing specific line numbers and the code flow you followed
- stating what happens at each step, not just the final result
- identifying unintended side effects even when the final output is correct

"Result is the same" does not mean "no finding." An unintended mutation (e.g., a query builder method that mutates the original object, an ordering clause that propagates to a subsequent aggregate) is a finding even if the end result happens to be identical. The question is whether the behavior is intentional, not whether the output is unchanged.

### Post-patch review discipline

After patching a finding, the patched code is new code. Review it as if someone else wrote it:

- do not carry "I just wrote this, so I know it works" bias into the next pass
- trace the patched code through the same mandatory activities as the original diff
- check whether the patch introduced new mutations, missing clones, or changed call semantics
- a patch that fixes finding F1 can introduce finding F4 — the review loop exists precisely for this

## Common Review Failures

These are not acceptable:

- skimming instead of reading every changed hunk
- dumping findings to the user instead of fixing them during the loop
- reporting “looks fine” without checking contracts and callers
- stopping after style issues while ignoring correctness risk
- saying “0 findings” after only one shallow pass
- trusting subagent or earlier review output without re-reading the final diff
- collapsing the entire checklist into “checked, all clear” without per-item evidence
- answering adversarial questions with “nothing” or “no issues” for every item
- reviewing your own code with less rigor because you wrote it and “know” it works
- patching a finding and moving to completion or verification without re-reviewing the patched diff
- treating “I fixed the issue” as equivalent to “the fix is verified” — the patch itself can introduce new problems
- treating “result is the same” as “no finding” — unintended mutations are findings regardless of output equivalence
- assuming one entry point covers all paths to the same behavior without verifying
- losing prior review findings after a revert-and-reimplement cycle
- running traces as checkbox items (“N/A”, “OK”, “checked”) instead of citing specific line numbers and code flows

## Zero-Findings Gate

Before declaring zero actionable findings on any review pass, confirm all of these:

1. every checklist item was explicitly checked against the diff — not batched, not skipped
2. every adversarial question was answered with a written response
3. error and failure paths were traced, not just the happy path
4. client-side and server-side state consistency was verified for stateful interactions
5. you did not review your own implementation with author bias — review as if someone else wrote it

If you cannot confirm all five, the review pass is incomplete. Go back and finish it.

A non-trivial diff where the first review pass finds zero findings is almost certainly under-reviewed. When this happens, re-read the diff with a different lens: trace an error path end-to-end, trace a malicious input end-to-end, or trace a state mutation end-to-end. If the second reading still finds nothing, the zero is credible.

## Severity

Treat findings as blockers by default.

Do not downgrade an issue just because:

- the code works locally
- the diff is small
- the reviewer is tired

If the issue is in scope and fixable now, fix it now.

## Parallel Specialist Review

When the diff is large (spanning 5+ files or 3+ subsystems) or touches security-sensitive, performance-critical, or data-migration code, consider dispatching parallel specialist subagent reviews before the main agent's final review pass.

### Available specialist lenses

Use only the lenses relevant to the current diff. Do not spawn specialists for areas the diff does not touch.

| Lens | Focus |
|------|-------|
| Security | auth, input sanitization, secrets exposure, injection vectors |
| Performance | unbounded loops, N+1 queries, missing indexes, memory leaks |
| API Contract | backward compatibility, schema changes, versioning |
| Data Consistency | migration safety, dual-write correctness, rollback paths |
| Test Coverage | untested branches, weak assertions, missing edge-case tests |

### Execution rules

- dispatch relevant specialists in parallel, each with a bounded scope (the diff + surrounding context)
- each specialist returns a findings list with severity and file references
- the main agent deduplicates findings across specialists — same issue reported by multiple specialists counts once but gains confidence
- the main agent patches all actionable specialist findings before the final whole-diff review pass
- specialist reviews do not replace the main agent's mandatory traces, checklist, or adversarial questions — they supplement them
- do not dispatch specialists for trivial diffs or when only one subsystem is affected

### Adaptive gating

If the same specialist lens produces zero findings for 5 consecutive tasks in the same repository, it may be skipped for similar future diffs. Re-enable it when the diff touches that specialist's domain again.

## Final Responsibility

Subagents may perform first-pass reviews for isolated slices, but the main agent must perform the final whole-diff review before completion.

The main agent must satisfy both of these before completion:

1. the review-and-patch loop reached zero actionable findings
2. a fresh final whole-diff review pass after the last patch also reached zero actionable findings

## Stage Exit

This stage is complete when both conditions above are met.

Exit marker: `## REVIEW COMPLETE — 0 FINDINGS`
