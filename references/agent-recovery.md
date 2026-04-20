# Agent Recovery Stage

## Purpose

Handle failures of the agent itself, not failures in the code being worked on. When the agent is looping, drifting from scope, exhausting context, or producing degraded output, this stage provides structured detection and recovery.

Use [debugging.md](debugging.md) for code bugs. Use this file for agent behavior problems.

## When To Use

Activate agent recovery when any of these symptoms appear:

- the same action is attempted 3+ times with no progress
- output quality visibly degrades (vague statements, incomplete traces, skipped steps)
- scope drift — working on files or features not in the current spec/plan
- context budget is in DEGRADING or POOR tier (see SKILL.md context management)
- a subagent returns incoherent or contradictory results
- the agent is about to retry a failed approach without changing strategy

## Detection Patterns

### Loop detection

The agent is stuck in a loop when:

- the same file is edited, reverted, and re-edited without a new hypothesis
- the same test fails after 3 different fix attempts with the same root cause
- the review loop stall detection (see review.md) has triggered
- a command is retried with identical arguments expecting different results

### Drift detection

The agent has drifted when:

- files being edited are not in the current plan's file scope
- the work no longer maps to any spec requirement
- the requirement ledger in the spec no longer matches what the plan or diff is covering
- "while I'm here" refactoring has started without spec authorization
- the agent is investigating a tangent that is interesting but not blocking

### Degradation detection

Output quality is degrading when:

- review traces become shallow ("looks fine", "no issues") after previously being detailed
- checklist items are batched as "all clear" instead of individually checked
- the agent stops citing specific line numbers and code paths
- responses become shorter and more vague over successive passes
- the agent starts making claims without running verification

## Recovery Procedure

### Step 1: Pause and diagnose

Stop the current action. Do not attempt another fix, edit, or review pass. Identify which failure pattern is active:

| Pattern | Primary cause | Recovery path |
|---------|--------------|---------------|
| Loop | wrong hypothesis or approach | Step 2: reset approach |
| Drift | lost connection to spec/plan | Step 3: re-anchor |
| Degradation | context exhaustion or fatigue | Step 4: compact and refocus |
| Subagent failure | bad delegation or scope | Step 5: re-scope subagent |

### Step 2: Reset approach (for loops)

1. stop the current approach entirely
2. re-read the error or failing test from scratch — do not rely on memory of what it said
3. form a new hypothesis that is materially different from prior attempts
4. if 3 materially different approaches have failed, escalate to user with:
   - what was tried (one line each)
   - the exact blocker
   - what information or decision would unblock progress

### Step 3: Re-anchor (for drift)

1. re-read the current spec and plan
2. compare current work against the plan's task list
3. identify what drifted and why
4. abandon out-of-scope work — do not save it "for later"
5. return to the last in-scope task in the plan

### Step 4: Compact and refocus (for degradation)

1. summarize current progress: what is done, what remains, what the current blocker is
2. if context budget is DEGRADING or POOR, suggest compaction at the current stage boundary
3. after compaction, re-read the spec, plan, and the reference file for the current stage
4. resume from the current task with fresh context

### Step 5: Re-scope subagent (for subagent failures)

1. read the subagent's full output — do not rely on its summary
2. identify whether the failure is:
   - scope too broad — the subagent tried to do too much
   - missing context — the subagent lacked information it needed
   - conflicting instructions — the prompt was ambiguous
3. re-dispatch with a narrower scope, additional context, or clarified instructions
4. if the same subagent fails twice on the same task, absorb the task into main agent execution

## Escalation

Escalate to the user when:

- recovery has been attempted and the agent is still stuck
- the blocker requires a decision outside the current spec
- the agent suspects the spec itself is the problem
- context is critically low and compaction alone will not help

Escalation format:

```
## Agent Recovery: Escalation

**Pattern:** [loop | drift | degradation | subagent failure]
**What happened:** [one sentence]
**What was tried:** [bullet list, one line each]
**What is needed:** [decision, information, or environment change]
```

## Red Flags

Stop and enter this recovery procedure if you catch yourself thinking:

- "let me try one more time with the same approach"
- "I'll just fix this unrelated thing while I'm here"
- "the details don't matter at this point"
- "I already know what the subagent found without reading its output"
- "I'll skip the traces this time since I wrote the code"

These are symptoms, not solutions. Enter recovery before the next action.
