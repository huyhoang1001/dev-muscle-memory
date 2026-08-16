# dev-muscle-memory

Rust-focused AI agent skills. Install with one command, works across 72+ agents.

[![skills.sh](https://skills.sh/b/huyhoang1001/dev-muscle-memory)](https://skills.sh/huyhoang1001/dev-muscle-memory)

## Install

```bash
npx skills add huyhoang1001/dev-muscle-memory
```

That's it for most agents. `npx skills` auto-detects which agents you have installed and drops skills in the right place.

**For Kiro specifically** — since Kiro runs as an IDE rather than a CLI binary, the auto-detector won't find it. Use `--copy -a kiro-cli` to install directly into `.kiro/skills/`:

```bash
npx skills add huyhoang1001/dev-muscle-memory --all -a kiro-cli --copy -y
```

### Options

```bash
# List available skills first
npx skills add huyhoang1001/dev-muscle-memory --list

# Install specific skills only
npx skills add huyhoang1001/dev-muscle-memory --skill rust-code-review --skill rust-dev

# Install globally (available in all projects)
npx skills add huyhoang1001/dev-muscle-memory -g

# Install to specific agents only
npx skills add huyhoang1001/dev-muscle-memory -a kiro-cli -a claude-code

# Non-interactive (CI/CD)
npx skills add huyhoang1001/dev-muscle-memory --all -y
```

### Keeping skills up to date

```bash
npx skills update
```

---

## Skills

### `rust-code-review`

Senior-level Rust code review. Covers what `rustc` **cannot** catch:

| Area | What it checks |
|------|---------------|
| Unsafe soundness | SAFETY comments, FFI contracts, transmute, invariants |
| Async hazards | Blocking in async, Mutex across `.await`, cancellation safety in `select!` |
| Ownership | Unnecessary clones, Arc overuse, Cow opportunities |
| Error handling | thiserror/anyhow split, unwrap in production, error chains |
| Performance | collect(), allocations, Box\<dyn\> in hot paths |
| API design | Idiomatic Rust, #[must_use], #[non_exhaustive], newtype pattern |

### `rust-dev`

Hands-on Rust development assistant:

- Compiler error triage — root cause, not just "add .clone()"
- Pattern selection — typestate, state machine, sealed traits, retry, cancellation tokens
- Crate recommendations — curated list by use case
- Project structure — workspace layout, feature flags, CI config

### `repo-explorer`

Maps any repository's architecture, modules, data flow, and conventions. Useful before making large changes to an unfamiliar codebase.

---

## Kiro agent configs (optional)

`npx skills` installs the skill instructions. For Kiro specifically, you can also install **agent configs** that pre-wire the skill, tool permissions, and a system prompt into a named agent — so you switch to it by name instead of typing a slash command.

```bash
# Copy agent configs into your project
curl -fsSL https://raw.githubusercontent.com/huyhoang1001/dev-muscle-memory/main/agents/kiro/rust-code-review.json \
  -o .kiro/agents/rust-code-review.json

curl -fsSL https://raw.githubusercontent.com/huyhoang1001/dev-muscle-memory/main/agents/kiro/rust-dev.json \
  -o .kiro/agents/rust-dev.json
```

Or just grab both at once:

```bash
mkdir -p .kiro/agents && \
  for agent in rust-code-review rust-dev; do
    curl -fsSL "https://raw.githubusercontent.com/huyhoang1001/dev-muscle-memory/main/agents/kiro/$agent.json" \
      -o ".kiro/agents/$agent.json"
  done
```

Then switch agents via the **agent picker in the bottom bar of the Kiro chat input**.

See [`agents/kiro/README.md`](./agents/kiro/README.md) for model selection and project-specific context setup.

---

## Adding project-specific context

The skills and agents work on any Rust project. To give them your project's conventions (hot path rules, MSRV, module layout), add a steering file:

```
.kiro/steering/conventions.md   ← your project rules
```

Then reference it in the agent JSON:

```json
{
  "resources": [
    "skill://.kiro/skills/rust-code-review/SKILL.md",
    "file://.kiro/steering/conventions.md"
  ]
}
```

---

## Structure

```
dev-muscle-memory/
├── skills/                     # Portable, platform-agnostic (npx skills installs these)
│   ├── rust-code-review/
│   │   ├── SKILL.md
│   │   └── references/         # Detailed checklists loaded on demand
│   ├── rust-dev/
│   │   ├── SKILL.md
│   │   └── references/
│   └── repo-explorer/
│       ├── SKILL.md
│       └── references/
└── agents/
    └── kiro/                   # Kiro-specific agent JSONs (optional, copy into .kiro/agents/)
        ├── rust-code-review.json
        ├── rust-dev.json
        └── README.md
```

Skills follow the [Agent Skills](https://agentskills.io/) open standard — compatible with Kiro, Claude Code, Cursor, Codex, and [30+ other agents](https://github.com/vercel-labs/skills#supported-agents).

---

## License

MIT
