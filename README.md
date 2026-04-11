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

## Workflow

```
 Explore ──▶ Clarify ──▶ Design ──▶ Spec ──▶ Plan ──▶ Execute ──▶ Review ──▶ Verify ──▶ Done
```

| Stage | What Happens |
|---|---|
| **Explore** | Read repository context, local guardrails (`AGENTS.md`, `CLAUDE.md`), and recent changes |
| **Clarify** | Ask only what prevents building the wrong thing — skip routine approvals |
| **Design** | Propose approaches with trade-offs, recommend one, collaborate with the user |
| **Spec** | Write a concrete spec capturing goal, scope, design, edge cases, and testing strategy |
| **Plan** | Decompose into executable tasks with file targets, test expectations, and verification steps |
| **Debug** | When the task involves a bug: investigate root cause with evidence before proposing fixes |
| **Execute** | Implement with TDD discipline — failing test first, minimal fix, verify green |
| **Review** | Harsh diff-based review loops until actionable findings reach zero; for implementation work, fix findings inside the loop before completion |
| **Verify** | Run fresh evidence-based verification — no "should work now" claims allowed |

The original user request is treated as authorization to continue once the spec is clear enough to write correctly. The agent stops only for real safety gates.

## Key Principles

- **Self-contained** — no external workflow skills needed; runtime tools and subagents are used directly
- **Spec and plan first** — non-trivial work always gets a written spec and implementation plan
- **TDD by default** — no production code without a failing test first (when a test surface exists)
- **Harsh review** — review the diff, not the codebase; for implementation work, absorb and patch findings inside the loop until zero actionable findings remain
- **Evidence over confidence** — verification must produce observable proof, not assertions
- **Review before completion** — passing tests does not replace the fresh final review pass
- **Minimal clarification** — ask only when the answer prevents building the wrong thing
- **Scope discipline** — no speculative refactoring, no unrelated cleanup

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
    └── verification.md             # Evidence-based verification and completion rules
```

### File Roles

- **`SKILL.md`** — The main contract. Defines when to use Superpilot, the full workflow, hard rules, trivial exceptions, storage paths, and safety gates.
- **`references/`** — Stage-specific guides. Each file covers how to execute one stage of the workflow in detail.
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
- Let `superpilot` own spec, plan, TDD, review, and verification flow.
- Once the work is clear enough for a correct spec, treat the original request
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
