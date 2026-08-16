# AGENTS.md

Guidance for AI coding agents working with code in this repository.

## Repository Overview

A collection of agent skills focused on Rust development. Skills follow the [Agent Skills](https://agentskills.io/) open standard and are installable via `npx skills add huyhoang1001/dev-muscle-memory`.

## Creating a New Skill

### Directory Structure

```
skills/
  {skill-name}/           # kebab-case, must match SKILL.md name field
    SKILL.md              # Required: frontmatter + instructions
    references/           # Optional: detailed docs loaded on demand
      {topic}.md
```

### SKILL.md Frontmatter Requirements

```yaml
---
name: skill-name          # lowercase, hyphens only, max 64 chars, matches folder name
description: "..."        # 1-1024 chars, include keywords and when-to-use context
license: MIT
metadata:
  author: huyhoang1001
  version: "1.0"
---
```

### Rules for Good Skills

- Keep `SKILL.md` under 500 lines — move detail into `references/`
- Always tell the agent what NOT to do (review-only, don't implement without confirmation, etc.)
- Reference files explicitly: `Load references/rust-unsafe.md for detailed SAFETY comment requirements`
- Write the description for activation matching — specific keywords beat clever prose

### Adding a New Skill

1. Create `skills/{name}/SKILL.md` with valid frontmatter
2. Add reference files to `skills/{name}/references/` if needed
3. Add metadata to `skills/{name}/metadata.json`
4. Add the skill to the appropriate group in `skills.sh.json`
5. If adding a Kiro agent config, add it to `agents/kiro/{name}.json`

## Repository Structure

```
dev-muscle-memory/
├── skills/               # Portable skills (npx skills installs these)
│   ├── rust-code-review/
│   ├── rust-dev/
│   └── repo-explorer/
├── agents/
│   └── kiro/             # Kiro-specific agent JSON configs
├── docs/                 # Blog posts and guides
├── skills.sh.json        # skills.sh grouping and metadata
├── AGENTS.md             # This file
└── README.md
```
