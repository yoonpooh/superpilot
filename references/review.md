# Review Stage

## Purpose

Review should be stricter than implementation. The goal is to prove the change is ready, not to feel good about it.

Review the diff until actionable findings reach zero.

## Review Scope

- run `git diff` on the actual changeset
- review only changed lines and their immediate calling context
- do not pad the review with unrelated unchanged code

## Review Loop

Repeat this loop:

1. capture the current diff
2. read every changed hunk
3. identify actionable findings
4. patch them
5. regenerate the diff
6. review again

Exit only when actionable findings are zero, unless a true safety gate or external blocker appears.

If the same finding repeats for 2 consecutive rounds and you cannot resolve the disagreement with evidence, treat it as a judgment conflict and escalate to the user.

## Required Procedure

For every review pass:

1. Run `git diff`
2. Read every changed line carefully
3. For each changed block, answer:
   - what are the possible inputs?
   - what edge cases hit this path?
   - what can fail here?
   - does the caller still match the contract?
   - is this consistent with nearby code patterns?
4. Apply the checklist below
5. Ask the adversarial questions below
6. Report findings with exact file references from the diff or changed files

## Checklist

Check each category explicitly against the diff:

- input validation
- error and exception handling
- return-value and caller compatibility
- regression risk to existing callers
- concurrency and ordering assumptions
- data and state consistency
- performance or resource blowups
- security and sensitive-data exposure
- test coverage for the most plausible failure mode
- actual alignment with the current spec
- operability and debuggability
- partial failure behavior

## Adversarial Questions

Before calling the review complete, answer all of these:

- If this ships as-is, what is most likely to break first?
- What incorrect assumption does this change make about inputs, ordering, state, or third-party behavior?
- What test would fail immediately if the implementation were subtly wrong in the most plausible way?
- If malicious or malformed input reaches this path, what happens?
- What would a strong reviewer challenge in this diff?

If any answer produces a credible problem, that is a finding.

## Review Mindset

Use a deployment-blocker mindset:

- assume the diff is not ready until it proves otherwise
- do not confuse familiarity with correctness
- do not stop after finding only surface issues
- if the review felt easier than the implementation, review again more deeply

## Common Review Failures

These are not acceptable:

- skimming instead of reading every changed hunk
- reporting “looks fine” without checking contracts and callers
- stopping after style issues while ignoring correctness risk
- saying “0 findings” after only one shallow pass
- trusting subagent or earlier review output without re-reading the final diff

## Severity

Treat findings as blockers by default.

Do not downgrade an issue just because:

- the code works locally
- the diff is small
- the reviewer is tired

If the issue is in scope and fixable now, fix it now.

## Final Responsibility

Subagents may perform first-pass reviews for isolated slices, but the main agent must perform the final whole-diff review before completion.
