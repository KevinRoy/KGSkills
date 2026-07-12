# Skill Format Checklist

Use this checklist when turning `skill-template` into a concrete skill.

## Required Files

- `SKILL.md`: required. Contains YAML frontmatter and concise instructions.
- `agents/openai.yaml`: recommended. Contains UI metadata, default prompt, policy, and optional MCP dependencies.

## SKILL.md Frontmatter

Use exactly these fields:

```yaml
---
name: skill-name
description: Clear explanation of what the skill does and when to use it.
---
```

Rules:

- Match `name` to the folder name.
- Use lowercase letters, digits, and hyphens.
- Put trigger conditions in `description`, not only in the body.
- Do not add extra frontmatter fields unless the platform requirements change.

## Body Pattern

Good default structure:

```markdown
# Skill Name

## Purpose

One or two sentences.

## Workflow

Step-by-step procedure.

## Resources

Tell Codex when to read specific reference files or use assets/scripts.

## Validation

Commands or checks to run before finishing.
```

## Resource Rules

- Put long domain details in `references/`.
- Put deterministic repeated code in `scripts/`.
- Put templates and reusable media in `assets/`.
- Link each reference from `SKILL.md` with a clear condition for when to read it.
- Keep reference files one level away from `SKILL.md`; avoid deep reference chains.

## Validation Command

```bash
python3 /Users/luozheng/.codex/skills/.system/skill-creator/scripts/quick_validate.py /path/to/skill-name
```
