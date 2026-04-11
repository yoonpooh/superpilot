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
3. Write a spec
4. Write a plan
5. Execute autonomously
6. Review the diff aggressively
7. Verify with evidence
8. Report completion

The original user request is treated as authorization to continue once the work is clear enough to describe correctly. The agent should stop only for real safety gates such as destructive actions, missing credentials, or contradictory requirements.

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
mkdir -p ~/.agents/skills
cp -R ./superpilot ~/.agents/skills/superpilot
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
```

## Scope

`superpilot` is for:

- bug fixes
- feature work
- refactoring tied to a real task
- config changes
- workflow and skill changes inside existing repositories

`superpilot` is not primarily for:

- greenfield project creation
- deployment orchestration by default
- commit or PR workflow by default

## License

MIT
