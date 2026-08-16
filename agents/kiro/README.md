# Kiro Agents

Ready-to-use agent configs for [Kiro](https://kiro.dev). These are the platform-specific counterpart to the skills in `../../skills/`.

## Model selection

The agent configs don't specify a model — they use whatever you've set as your Kiro default. To use a specific model for an agent, add a `"model"` field:

```json
{
  "model": "claude-sonnet-4"
}
```

Use `/model` in Kiro chat to see available model IDs on your account.

## How to use

### Option A — Copy into your project (project-scoped)

```bash
cp rust-code-review.json /your-project/.kiro/agents/
cp rust-dev.json /your-project/.kiro/agents/
```

Then switch to the agent via the agent picker in Kiro's chat input bar.

### Option B — Copy to global agents (available in all projects)

```bash
cp rust-code-review.json ~/.kiro/agents/
cp rust-dev.json ~/.kiro/agents/
```

## Prerequisites

The agent JSONs reference `skill://.kiro/skills/rust-code-review/SKILL.md` — so the skills need to be imported first.

Import via Kiro panel → **Agent Steering & Skills** → **+** → **Import** → **GitHub**:

| Skill | URL |
|-------|-----|
| `rust-code-review` | `https://github.com/huyhoang1001/dev-muscle-memory/tree/main/skills/rust-code-review` |
| `rust-dev` | `https://github.com/huyhoang1001/dev-muscle-memory/tree/main/skills/rust-dev` |
| `repo-explorer` | `https://github.com/huyhoang1001/dev-muscle-memory/tree/main/skills/repo-explorer` |

## Adding project-specific context

The base agents work on any Rust project. To add your project's conventions (hot path rules, MSRV, module layout, etc.), extend the agent JSON with a `resources` entry pointing to a steering file:

```json
{
  "resources": [
    "skill://.kiro/skills/rust-code-review/SKILL.md",
    "file://.kiro/steering/structure.md",
    "file://.kiro/steering/tech.md"
  ]
}
```

See `heap-sentry` example in the repo root for a full project-specific setup.
