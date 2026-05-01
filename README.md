<p align="center">
  <h1 align="center">Superpilot</h1>
  <p align="center">
    A self-contained workflow skill that takes coding agents from design through verification — no external skill packs required.
  </p>
  <p align="center">
    <a href="README.md">English</a> | <a href="README.ko.md">한국어</a>
  </p>
</p>

---

## What is Superpilot?

Superpilot is a standalone workflow skill for AI coding agents working in existing repositories. It defines a disciplined, end-to-end process — from collaborative design to autonomous execution to harsh self-review — all within a single skill.

Most agent workflow systems rely on stacking multiple external skills together. Superpilot takes a different approach: the entire process lives in one repository and requires no external dependencies.

The design target is zero-intervention completion: the agent should research when facts matter, preserve the full requested scope, choose its own execution topology, and keep going until the work is actually verified.

## Workflow

```
 Explore ──▶ Clarify ──▶ Design ──▶ Spec ──▶ Plan ──▶ Execute ──▶ Review ──▶ Verify ──▶ Done
```

| Stage | What Happens |
|---|---|
| **Explore** | Read repository context, local guardrails (`AGENTS.md`, `CLAUDE.md`), recent changes, baseline state, and whether isolated execution is needed |
| **Clarify** | Present code-based assumptions for the user to correct, rather than asking open-ended questions |
| **Design** | Research when needed, challenge framing and hidden scope, propose approaches with trade-offs, recommend one, collaborate with the user |
| **Spec** | Write a concrete spec capturing goal, requirement ledger, design, execution strategy, edge cases, and testing strategy |
| **Plan** | Decompose into executable tasks with file targets, ownership, workspace strategy, requirement coverage, and verification steps |
| **Debug** | When the task involves a bug: investigate root cause with evidence before proposing fixes |
| **Execute** | Implement with TDD discipline — failing test first, minimal fix, verify green. Pre-edit investigation gate ensures files are read before edited. Test failures are triaged as in-branch, pre-existing, or flaky before investigation. Workspace isolation and fresh subagents are used when they improve autonomy and clarity. GREEN triggers a mandatory transition to Review, not completion |
| **Review** | Harsh diff-based review loops with stall detection (3 consecutive non-decreasing findings trigger escalation). Requirement-preservation, schema/contract drift, threat-model, UX, and DevEx lenses can all become findings. Optional parallel specialist reviews (Security, Performance, API Contract, Data Consistency, Test Coverage, UX/Design, DevEx) supplement the main review loop. Every patch requires a fresh re-review |
| **Verify** | 4-level verification (Exists → Substantive → Wired → Functional) with stub detection. User-flow, onboarding/DX, migration/codegen, and abuse-path checks are required when relevant. No "should work now" claims allowed |

For explicit implementation requests, the original user request is treated as authorization to continue once must-have requirements are user-confirmed or code-proven and the spec is clear enough to write correctly. The agent stops only for real safety gates.

## Key Principles

- **Self-contained** — no external workflow skills needed; runtime tools and subagents are used directly
- **Zero-intervention by default** — routine execution does not wait for approval; the agent keeps moving unless a real safety gate or material ambiguity appears
- **Research before guessing** — primary-source lookup is part of the workflow when external facts or library behavior matter
- **Spec and plan first** — non-trivial work always gets a written spec and implementation plan
- **Micro Task Mode** — low-risk, tightly scoped tasks skip the full workflow and use the smallest direct edit plus the smallest relevant check
- **Confirmed requirements first** — full workflow specs start only after must-have requirements are user-confirmed or code-proven
- **Requirement-ledger discipline** — requested outcomes are tracked from user request through verification so scope cannot quietly shrink
- **TDD by default** — no production code without a failing test first (when a test surface exists)
- **Harsh review** — review the diff, not the codebase; for implementation work, absorb and patch findings inside the loop until zero actionable findings remain
- **Evidence over confidence** — verification must produce observable proof, not assertions
- **Review before completion** — GREEN tests trigger review, not completion. Every patch requires a fresh re-review pass before exit
- **Execution topology is deliberate** — the agent chooses in-place work, isolated worktrees, or fresh subagents based on task risk and context budget
- **State trace discipline** — when changes affect visible or persisted state, review must trace divergence, correction, and boundary behavior explicitly
- **Real workflow proof** — user-facing, onboarding, and DX changes are not considered done without an actual walkthrough or strongest equivalent proof
- **Drift detection** — schema, contract, generated artifact, fixture, and docs drift are treated as real defects, not cleanup
- **Assumptions-first clarification** — analyze codebase first, present assumptions with confidence levels, let user correct only what is wrong
- **Scope discipline** — no speculative refactoring, no unrelated cleanup
- **Pre-edit investigation** — every file must be read and its callers identified before editing; no edits based on assumptions
- **Stage transition markers** — each stage has a defined exit marker that must be satisfied before advancing
- **Context budget awareness** — behavior adapts across PEAK/GOOD/DEGRADING/POOR tiers to maintain output quality
- **Context-rot resistance** — fresh subagent contexts, compact stage summaries, and bounded research reduce quality decay on long tasks
- **Agent self-recovery** — structured detection and recovery for agent loops, scope drift, and context degradation
- **Test failure triage** — classify failures as in-branch, pre-existing, or flaky before spending time debugging

## Repository Layout

```
superpilot/
├── SKILL.md                        # Skill contract — workflow, rules, safety gates
├── LICENSE
├── agents/
│   └── openai.yaml                 # OpenAI Codex agent interface definition
└── references/
    ├── spec.md                     # Collaborative design and spec writing
    ├── plan.md                     # Implementation planning and subagent split rules
    ├── debugging.md                # Root-cause investigation procedure
    ├── implementation.md           # TDD, execution discipline, blocker handling
    ├── review.md                   # Harsh diff review procedure and checklist
    ├── verification.md             # Evidence-based verification and completion rules
    └── agent-recovery.md           # Agent loop, drift, and degradation recovery
```

### File Roles

- **`SKILL.md`** — The main contract. Defines when to use Superpilot, Micro Task Mode, the full workflow, hard rules, storage paths, and safety gates.
- **`references/`** — Stage-specific guides. Each file covers how to execute one stage of the workflow in detail. Includes agent self-recovery for handling agent-level failures.
- **`agents/openai.yaml`** — Agent interface for OpenAI Codex integration.

## Installation

### 1. Shared Skill Source

Copy Superpilot to a shared location that multiple agent runtimes can reference:

```bash
mkdir -p ~/.agents/skills/superpilot
rsync -a ./ ~/.agents/skills/superpilot/
```

### 2. Claude Code

Symlink from the shared source:

```bash
mkdir -p ~/.claude/skills
ln -s ~/.agents/skills/superpilot ~/.claude/skills/superpilot
```

### 3. Codex

Codex may only index real directories under `~/.codex/skills`, so use a copy instead of a symlink:

```bash
mkdir -p ~/.codex/skills
cp -R ~/.agents/skills/superpilot ~/.codex/skills/superpilot
```

## Recommended Agent Configuration

Add a rule to your agent instruction file to make Superpilot the default workflow:

```md
# AGENTS.md or CLAUDE.md

- For non-trivial work in existing repositories, use the `superpilot` skill first.
- For low-risk, tightly scoped tasks, use Micro Task Mode instead of the full workflow.
- Before writing a full-workflow spec, confirm every must-have requirement or prove it from repository evidence.
- Let `superpilot` own research, spec, plan, TDD, review, and verification flow.
- Let `superpilot` preserve requirement coverage and choose isolated worktrees or subagents when autonomy benefits from them.
- For explicit implementation requests, once must-have requirements are confirmed or proven and the work is clear enough for a correct spec, treat the original request
  as authorization to continue unless a safety gate is triggered.
```

## Scope

### Designed For

- Bug fixes
- Feature work
- Refactoring tied to a real task
- Config, CI, or workflow changes
- Test additions or test repair
- Review-only audits of existing diffs, branches, commits, or pull requests

### Not Designed For

- Greenfield project creation
- Deployment orchestration (by default)
- Commit or PR workflow (by default)

## Specs and Plans Storage

Superpilot writes specs and plans to a local directory:

```
~/.superpilot/docs/<repo-name>/
├── specs/    # YYYY-MM-DD-<short-slug>.md
└── plans/    # YYYY-MM-DD-<short-slug>.md
```

`<repo-name>` is the git root basename. Slugs describe the task, not the branch.

## Safety Gates

The agent stops and asks the user only when:

- An action is destructive or irreversible
- Credentials, secrets, or access are missing
- Requirements are contradictory or newly unsafe

Routine execution, planning, and implementation choices do not require user approval.

## License

[MIT](LICENSE)
