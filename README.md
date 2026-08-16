# dev-muscle-memory

Personal AI agent skills and configs for Rust development. Built to work with Kiro today, structured to extend to Claude, Cursor, or any other agentic tool tomorrow.

## Structure

```
dev-muscle-memory/
├── skills/                        # Portable, platform-agnostic prompt packages
│   ├── rust-code-review/          # Rust code review — unsafe, async, ownership, perf
│   ├── rust-dev/                  # Rust dev assistant — patterns, crates, tooling
│   └── repo-explorer/             # Repo mapping — architecture, modules, data flow
└── agents/                        # Platform-specific agent configs
    └── kiro/                      # Kiro IDE/CLI agent JSONs
        ├── rust-code-review.json
        └── rust-dev.json
```

**Skills** are portable instruction packages following the [Agent Skills](https://agentskills.io/) standard. They contain the actual knowledge — checklists, patterns, reference guides. They work anywhere: Kiro, Claude Projects, any compatible tool.

**Agents** are platform-specific wrappers. They define model, tools, permissions, and wire in the skills + project context. Adding support for a new platform means adding a new folder under `agents/` — skills stay untouched.

---

## Skills

### [`skills/rust-code-review`](./skills/rust-code-review/)

Senior-level Rust code review. Covers what the compiler **cannot** catch:

- **Unsafe soundness** — SAFETY comments, FFI contracts, transmute, invariants
- **Async hazards** — blocking in async, Mutex across `.await`, cancellation safety in `select!`
- **Ownership** — unnecessary clones, Arc overuse, Cow opportunities
- **Error handling** — thiserror/anyhow split, unwrap in production, error chains
- **Performance** — collect(), allocations, Box\<dyn\> in hot paths
- **API design** — idiomatic Rust, #[must_use], #[non_exhaustive], newtype pattern

### [`skills/rust-dev`](./skills/rust-dev/)

Hands-on Rust development assistant:

- Compiler error triage — root cause, not just "add .clone()"
- API design — types, error design, builder patterns
- Pattern selection — typestate, state machine, sealed traits, retry, cancellation tokens
- Crate recommendations — curated list by use case
- Project structure — workspace layout, feature flags, CI config

### [`skills/repo-explorer`](./skills/repo-explorer/)

Maps any repository's architecture, modules, data flow, dependencies, and conventions. Produces a structured map to navigate an unfamiliar codebase confidently.

---

## Setup

### Step 1 — Import skills into Kiro

Skills live in `.kiro/skills/` (project) or `~/.kiro/skills/` (global).

Import via **Kiro panel → Agent Steering & Skills → + → Import → GitHub**:

| Skill | URL |
|-------|-----|
| `rust-code-review` | `https://github.com/huyhoang1001/dev-muscle-memory/tree/main/skills/rust-code-review` |
| `rust-dev` | `https://github.com/huyhoang1001/dev-muscle-memory/tree/main/skills/rust-dev` |
| `repo-explorer` | `https://github.com/huyhoang1001/dev-muscle-memory/tree/main/skills/repo-explorer` |

Once imported, type `/rust-code-review` in chat to invoke as a slash command.

### Step 2 — Install agent configs (optional but recommended)

Agent configs pre-wire the skills, model, tool permissions, and a system prompt into a named agent you switch to by name — no manual context loading per session.

```bash
# Project-scoped (just this repo)
cp agents/kiro/rust-code-review.json .kiro/agents/
cp agents/kiro/rust-dev.json .kiro/agents/

# Or global (all projects)
cp agents/kiro/rust-code-review.json ~/.kiro/agents/
cp agents/kiro/rust-dev.json ~/.kiro/agents/
```

Then switch agents via the **agent picker in the bottom bar of the Kiro chat input**.

### Step 3 — Add project context (recommended)

The base agents work on any Rust project. For project-specific rules (hot path constraints, MSRV, module conventions), add a steering file and reference it in the agent JSON:

```
.kiro/steering/skills.md    ← your project's conventions for the agent
```

```json
{
  "resources": [
    "skill://.kiro/skills/rust-code-review/SKILL.md",
    "file://.kiro/steering/structure.md",
    "file://.kiro/steering/skills.md"
  ]
}
```

See the `heap-sentry` project for a complete example.

---

## Extending to other platforms

To add Claude, Cursor, or another tool:

```
agents/
├── kiro/           ✓ done
├── claude/         → Claude Projects instructions + tool configs
└── cursor/         → .cursorrules or cursor agent format
```

Skills in `skills/` never change — only the platform adapter in `agents/` gets added.

---

## Customizing

Reference files in `skills/*/references/` are plain Markdown — fork and edit to match your team's conventions:

| File | What to customize |
|------|------------------|
| `rust-code-review/references/rust-unsafe.md` | Project-specific unsafe invariants |
| `rust-code-review/references/rust-async.md` | Async runtime conventions |
| `rust-dev/references/rust-crates.md` | Your standardized crate list |
| `rust-dev/references/rust-project-structure.md` | Your workspace layout |

---

## License

MIT
