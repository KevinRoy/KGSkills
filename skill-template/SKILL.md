---
name: skill-template
description: Standard scaffold for creating reusable Codex skills, including personal skills, company-internal skills, third-party workflow skills, and skills that declare MCP tool dependencies. Use when building, copying, reviewing, or standardizing a Codex skill folder with SKILL.md, agents/openai.yaml, optional references, scripts, and assets.
---

# Skill Template

## Purpose

Use this template as the starting point for new skills. Keep the skill focused on instructions another Codex instance needs in order to perform a repeatable task well.

Prefer concise procedural guidance in `SKILL.md`. Move detailed schemas, policies, API notes, examples, and long references into `references/`.

## Create a New Skill

1. Copy this folder and rename it to the final skill name.
2. Use lowercase letters, digits, and hyphens only, such as `company-pr-review`, `personal-writing-style`, or `notion-research-sync`.
3. Update the frontmatter:
   - `name`: match the folder name exactly.
   - `description`: explain what the skill does and when Codex should use it. Include trigger contexts here because Codex reads this before loading the body.
4. Replace this body with the workflow, rules, references, and validation steps for the target skill.
5. Update `agents/openai.yaml` so the UI metadata and tool dependencies match the target skill.
6. Run the skill validator before using or publishing the skill.

## Recommended Layout

```text
skill-name/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   ├── skill-format.md
│   └── mcp-dependencies.md
├── scripts/
└── assets/
```

Use only the directories the skill actually needs:

- `references/`: detailed docs, API notes, schemas, policies, long examples, company knowledge.
- `scripts/`: repeatable operations that should be deterministic, tested, or too verbose to rewrite each time.
- `assets/`: templates, icons, fonts, images, starter projects, documents, or other files used in outputs.

## Writing SKILL.md

Write instructions in imperative form. Include:

- A short purpose section.
- A workflow or decision tree for the common path.
- Specific rules that are easy to forget.
- Pointers to reference files with clear conditions for when to read each one.
- Validation steps, especially for scripts, generated artifacts, or external integrations.

Avoid:

- Generic advice Codex already knows.
- Long copied documentation in `SKILL.md`.
- Extra files such as `README.md`, install guides, changelogs, or quick references unless the target platform explicitly requires them.

## MCP Support

Declare MCP dependencies in `agents/openai.yaml` under `dependencies.tools`. Read `references/mcp-dependencies.md` when creating or updating a skill that relies on MCP servers or connector tools.

Keep MCP instructions practical:

- Name the MCP server or connector the skill expects.
- Explain what the tool is used for.
- Include authentication, workspace, or environment assumptions only when they are required.
- Provide a fallback path when the MCP dependency is unavailable.

## Reference Files

Read `references/skill-format.md` when you need a checklist for converting this template into a real skill.

Read `references/mcp-dependencies.md` when you need to add, remove, or document MCP tool dependencies.

## Validation

After editing a real skill, run:

```bash
python3 /Users/luozheng/.codex/skills/.system/skill-creator/scripts/quick_validate.py /path/to/skill-name
```

Fix any frontmatter, naming, or structure issues before relying on the skill.
