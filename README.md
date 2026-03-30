# Skills

A collection of global agent skills that can be installed locally on any machine.

## What are skills?

Skills are reusable instructions for AI coding agents (such as GitHub Copilot). Each skill is a Markdown file (`.md`) that describes a specific capability, workflow, or set of guidelines the agent should follow.

## Where to add skills

Place each skill as a `.md` file directly in the root of this repository. Use a clear, descriptive file name that reflects the purpose of the skill, for example:

```
skills/
├── README.md
├── code-review.md
├── commit-messages.md
└── testing-guidelines.md
```

## How to add a skill

1. Create a new Markdown file in the root of this repository.
2. Give it a descriptive name using lowercase letters and hyphens (e.g. `my-skill.md`).
3. Write the skill content in plain Markdown. Describe the behavior, rules, or steps the agent should follow.

### Skill template

```markdown
# Skill Name

A short description of what this skill does.

## Guidelines

- Guideline one
- Guideline two
- Guideline three
```

## Installation

This repository is designed to be cloned into your global agent skills folder so that the skills are available across all projects on your machine. Refer to your agent's documentation for the exact path where global skills should be placed.
