# Debugging Stage

## Purpose

Do not guess at fixes. When the task is a bug, a failing test, a build failure, or unexpected behavior, find the root cause before implementing changes.

## Iron Law

```text
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you have not completed the investigation phase, you are not ready to propose or implement a fix.

## When To Use

Use this stage whenever the problem is not already fully explained by the current written design:

- failing tests
- broken builds
- runtime crashes
- incorrect behavior
- flaky behavior
- integration failures
- performance regressions

## Investigation Procedure

1. Read the full error or symptom carefully.
2. Reproduce it consistently.
3. Identify recent changes or nearby risk areas.
4. Trace the data and control flow backward to the source.
5. Form one concrete hypothesis.
6. Test the hypothesis with the smallest useful check.
7. Only after the root cause is supported by evidence should you move to implementation.

## Required Behaviors

### Read the evidence completely

- read stack traces fully
- capture exact error text
- note file paths, line numbers, error codes, and failing commands
- do not skip warnings just because a louder error also exists

### Reproduce before fixing

- identify exact steps
- confirm whether the issue is deterministic or flaky
- if it is flaky, gather more evidence rather than guessing

### Trace the flow

When the failure happens deep in a call chain or multi-component system, map:

- where the bad value or state originates
- where it is transformed
- where it is stored or restored
- where it is finally observed as a failure

For multi-layer systems, add temporary instrumentation at boundaries if needed to identify the failing layer.

### Hypothesis discipline

- form one hypothesis at a time
- make one bounded change or diagnostic to test it
- if it fails, return to investigation instead of stacking random fixes

## Root-Cause Red Flags

Stop and return to investigation if you catch yourself thinking:

- “just try this quick fix”
- “it is probably X”
- “let me change three things at once”
- “the issue seems obvious”
- “I already know the cause without reproducing it”

These are guessing patterns, not debugging.

## Transition To Implementation

Once the root cause is supported by evidence:

- capture the root cause in the plan or working notes
- define the failing test or repro that will prove the fix
- move to `implementation.md`
