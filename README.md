# Superpilot

`superpilot` is a standalone workflow skill for existing repositories.

It is designed for coding agents that should:

- clarify only what is materially ambiguous
- write a spec and plan for non-trivial work
- use TDD when there is a real test surface
- use subagents only when task independence is clear
- run harsh diff-based review loops
- verify with fresh evidence before claiming completion

Unlike workflow stacks that depend on many external skills, `superpilot` is built to be self-contained. The skill can still use runtime tools and subagents, but its core process lives in this repository.

## Philosophy

`superpilot` is optimized for this workflow:

1. Explore repository context
2. Clarify only what is necessary
3. Collaborate on the design
4. Write a spec
5. Write a plan
6. Execute autonomously
7. Review the diff aggressively
8. Verify with evidence
9. Report completion

The original user request is treated as authorization to continue once the work is clear enough to describe correctly. The agent should stop only for real safety gates such as destructive actions, missing credentials, or contradictory requirements.

Explicit review-only requests are handled differently: `superpilot` should inspect the diff and return findings without patching unless fixes are explicitly requested.

## Repository Layout

```text
superpilot/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── debugging.md
    ├── implementation.md
    ├── plan.md
    ├── review.md
    ├── spec.md
    └── verification.md
```

## Install

### Shared skill source

```bash
mkdir -p ~/.agents/skills/superpilot
rsync -a ./ ~/.agents/skills/superpilot/
```

### Claude Code

```bash
mkdir -p ~/.claude/skills
ln -s ~/.agents/skills/superpilot ~/.claude/skills/superpilot
```

### Codex

Codex may index only real directories under `~/.codex/skills`, so prefer a copy instead of a symlink there.

```bash
mkdir -p ~/.codex/skills
cp -R ~/.agents/skills/superpilot ~/.codex/skills/superpilot
```

## Recommended Agent Rule

Use `superpilot` as the primary workflow source of truth for non-trivial work in existing repositories.

Example:

```md
# AGENTS.md or CLAUDE.md

- For non-trivial work in existing repositories, use the `superpilot` skill first.
- Let `superpilot` own spec, plan, TDD, review, and verification flow.
- Once the work is clear enough for a correct spec, treat the original request as authorization to continue unless a safety gate is triggered.
```

## Scope

`superpilot` is for:

- bug fixes
- feature work
- refactoring tied to a real task
- config changes
- workflow and skill changes inside existing repositories
- review-only audits of an existing diff, branch, commit, or pull request

`superpilot` is not primarily for:

- greenfield project creation
- deployment orchestration by default
- commit or PR workflow by default

## License

MIT
