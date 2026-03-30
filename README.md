# Skills

A collection of personal agent skills for GitHub Copilot and compatible AI coding agents.

## What are skills?

Agent skills are folders of instructions, scripts, and resources that Copilot can load when relevant to improve its performance on specialized tasks. Copilot decides when to use a skill based on the task prompt and the skill's description.

For more information, see the [GitHub Copilot skills documentation](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-skills).

## Structure of a skill

Each skill lives in its own subdirectory and must contain a `SKILL.md` file. Additional resources (scripts, examples, supplementary Markdown files) can be added alongside it.

```
skills/
├── README.md
├── github-actions-failure-debugging/
│   └── SKILL.md
├── code-review/
│   └── SKILL.md
└── commit-messages/
    ├── SKILL.md
    └── examples.md
```

Subdirectory names must be **lowercase with hyphens** (e.g. `my-skill`).

## How to add a skill

1. Create a subdirectory with a lowercase, hyphen-separated name (e.g. `my-skill`).
2. Inside that directory, create a `SKILL.md` file.
3. Add YAML frontmatter and a Markdown body to `SKILL.md`.
4. Optionally, add scripts or other resources to the directory and reference them in the instructions.

### `SKILL.md` template

```markdown
---
name: my-skill
description: Brief description of what this skill does and when Copilot should use it.
---

Instructions, guidelines, and examples for Copilot to follow when this skill is active.
```

**Frontmatter fields:**

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Unique identifier; lowercase with hyphens. Should match the directory name. |
| `description` | Yes | What the skill does and when Copilot should use it. |
| `license` | No | License that applies to this skill. |

## Where skills are loaded from

Skills can be stored in two locations:

- **Personal skills** (shared across all projects): `~/.copilot/skills/`, `~/.claude/skills/`, or `~/.agents/skills/`
- **Project skills** (specific to one repository): `.github/skills/`, `.claude/skills/`, or `.agents/skills/` inside the repo

## Installation

To use these skills as personal skills, clone this repository into one of the supported personal skills directories:

```bash
git clone https://github.com/Ferre-Laridon/skills ~/.agents/skills
```

## Skills vs. custom instructions

Use **custom instructions** for simple rules that apply to almost every task (e.g. coding standards). Use **skills** for more detailed, task-specific instructions that Copilot should only load when relevant.
