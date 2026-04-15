# .claude

Personal AI configuration, rules, and skills for Claude CLI and Cursor.

## Contents

| Path | Purpose |
|---|---|
| `CLAUDE.md` | Global behavioral guidelines — loaded automatically in every conversation |
| `skills/` | Reusable agent skills for NCS development, Git, QA, PM, and writing workflows |

## Skills

Skills live under `skills/` as named directories, each containing a `SKILL.md` entry point.
Invoke a skill by mentioning its name in your request, e.g. *"use chsh-git-commit to commit these changes"*.

See individual `SKILL.md` files for usage details.
