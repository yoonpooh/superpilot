# Review Stage

## Purpose

Review should be stricter than implementation. The goal is to prove the change is ready, not to feel good about it.

Review the diff until actionable findings reach zero.

By default, review is an internal quality gate, not a user-facing report.

If the task itself is review-only, this stage becomes the primary deliverable:

- inspect the diff with the same harsh standard
- return findings ordered by severity
- do not patch unless the user explicitly asks for fixes

## Review Scope

- capture the actual changeset, not only plain `git diff`
- review tracked changes, staged changes, and any untracked files that are part of the task
- plain `git diff` alone is not sufficient when the work includes new untracked files
- if a new file is part of the work, either read it directly during review or stage it before the final whole-diff review pass
- review only changed lines and their immediate calling context
- do not pad the review with unrelated unchanged code

## Mandatory Trace Activities

These are not checklist items to glance at. They are concrete activities that require reading and following actual code paths in the diff. Do all that apply.

### 1. Failure path trace

For every external call, network request, or operation that can fail in the diff:

- identify every failure branch (exception, error response, timeout, empty result)
- trace each failure from the point of origin through to what the user sees
- verify that UI state, session state, and persisted state all stay consistent on failure
- if the failure is swallowed or only partially handled, that is a finding

### 2. State consistency trace

For every place the diff updates state in two or more locations (e.g., DOM and server, client array and session, cache and database):

- trace what happens when the operation succeeds
- trace what happens when the operation fails partway through
- verify that the two states cannot diverge
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

If a trace activity does not apply to the current diff (e.g., no external calls, no new endpoints), skip it and state why it does not apply.

## Review Loop

Repeat this loop:

1. capture the current diff
2. read every changed hunk
3. run mandatory trace activities (see below) — these are concrete code-path traces, not checkbox items
4. walk through every checklist item against every changed block — record what you checked and what you found per item
5. answer every adversarial question — record each answer
6. collect actionable findings
7. patch them
8. regenerate the diff
9. review again from step 1

Steps 3, 4, and 5 are not optional. A review pass that skips any of these is not a valid review pass and does not count toward the zero-findings exit condition.

Exit only when actionable findings are zero, unless a true safety gate or external blocker appears.

Completion is stricter than "I patched the last finding":

- after the final patch, run at least one fresh whole-diff review pass with no new code changes in between
- the fresh pass must also walk through every checklist item and adversarial question
- only if that fresh pass also returns zero actionable findings may the task move to final verification and completion
- if a later "code review" request on the same finished diff would obviously find an issue, the task was not actually complete

If the same finding repeats for 2 consecutive rounds and you cannot resolve the disagreement with evidence, treat it as a judgment conflict and escalate to the user.

Treat findings as internal work items:

- patch them immediately when they are in scope and fixable now
- do not stop to ask the user what to do with normal review findings
- do not present a findings list mid-task unless the user explicitly asked for a review-only outcome

## Required Procedure

For every review pass:

1. Capture the full changeset: `git diff`, `git diff --cached`, and any untracked task files via `git ls-files --others --exclude-standard`
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
- data and state consistency — can client-side and server-side state diverge? (e.g., optimistic UI without rollback)
- performance or resource blowups — can this path be called in an unbounded loop or without rate limiting?
- security and sensitive-data exposure — are secrets, PII, or internal details leaked in logs, responses, or error messages?
- test coverage for the most plausible failure mode — is there a test for the first thing that would break?
- actual alignment with the current spec — does the implementation match what was specified, not just "work"?
- operability and debuggability — can you diagnose a production failure from logs alone?
- partial failure behavior — if this operation half-succeeds, what state is left behind?
- dead code and unused parameters — are there parameters validated but never sent, or code paths that can never execute?

## Adversarial Questions

Before calling the review complete, answer all of these explicitly — write out each answer:

- If this ships as-is, what is most likely to break first?
- What incorrect assumption does this change make about inputs, ordering, state, or third-party behavior?
- What test would fail immediately if the implementation were subtly wrong in the most plausible way?
- If malicious or malformed input reaches this path, what happens?
- What would a strong reviewer challenge in this diff?
- What would a user encounter on the first error or edge-case path?

If any answer produces a credible problem, that is a finding.

Do not answer these questions in your head. Write out each answer as part of the review work. A one-word "nothing" for every question is a red flag that the review was too shallow.

## User Communication Rule

Default behavior:

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

## Final Responsibility

Subagents may perform first-pass reviews for isolated slices, but the main agent must perform the final whole-diff review before completion.

The main agent must satisfy both of these before completion:

1. the review-and-patch loop reached zero actionable findings
2. a fresh final whole-diff review pass after the last patch also reached zero actionable findings
