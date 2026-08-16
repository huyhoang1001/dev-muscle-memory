# `npx skills`: The Package Manager That Agent Context Deserved

There's a moment in the lifecycle of every useful abstraction where someone builds the tooling around it, and the thing stops being an interesting idea and starts being infrastructure.

For npm packages, that moment was npm itself. For Docker images, it was Docker Hub. For Homebrew formulas, it was the tap system. For agent skills, it's `npx skills`.

This is the story of where that tool came from, what it actually does, and why the open ecosystem it sits inside is worth paying attention to.

---

## Where It Started

The origin is worth knowing because it shapes how the tool thinks.

[Shu Ding](https://x.com/shuding), an engineer at Vercel, knows more about React and the browser internals than most people will ever discover. He sat down one weekend and wrote it all down — a kind of comprehensive React bible, encoding years of hard-won expertise into structured Markdown.

The question was how to ship it. A blog post might eventually feed into future model training, but you'd be waiting for Claude Sonnet 8 or GPT-9 to see the benefit. An MCP server felt too heavy for what was essentially a collection of Markdown documents. Agent Skills — the open standard for portable instruction packages — was the right format.

But then came the friction. Installing the skill meant copying files into slightly different directories for Cursor, Claude Code, Codex, OpenCode, and every other agent. The paths were different. The conventions were different. You were doing the same thing twelve times with twelve variations.

So Andrew Qu at Vercel built a CLI to handle it. That became `npx skills`. They added telemetry to surface what skills people were actually installing, which became the leaderboard at [skills.sh](https://skills.sh/). The whole thing went from idea to production in days.

Vercel CTO Malte Ubl described it cleanly: it's a package manager for agent context.

As of mid-2026, the repository has 29,000 stars, 136 contributors, 2.5k forks, and supports 72 agents. In seven months it went from a weekend project to infrastructure that major engineering teams depend on.

---

## The Problem It Solves

Before `npx skills`, distributing agent context was a manual, platform-specific operation. You wrote good instructions, and then you:

- Figured out where Kiro expects skill files (`.kiro/skills/`)
- Figured out where Claude Code expects them (`.claude/skills/`)
- Figured out where Cursor expects them (`.cursor/skills/`)
- Repeated this for however many agents your team uses
- Did it again when someone joined the team
- Did it again when you started a new project
- Hoped nobody forgot a step

This is the copy-paste problem. The knowledge was portable in theory — Agent Skills is an open format — but the installation wasn't. Every agent had its own path, its own conventions, its own edge cases.

`npx skills` is the abstraction over that. It knows where 72 agents expect their skills. It detects which agents you have installed. It handles the path logic so you write and publish once, and anyone on any supported agent can install with a single command.

---

## The Standard It Builds On

`npx skills` is tooling around the [Agent Skills specification](https://agentskills.io/), an open standard originally developed at Anthropic and now adopted by 40+ products including GitHub Copilot, VS Code, Cursor, Codex, Gemini CLI, Goose, Kiro, Mistral Vibe, Snowflake Cortex Code, and Databricks Genie Code.

The format is intentionally minimal. A skill is a directory containing a `SKILL.md` file:

```
my-skill/
├── SKILL.md          # Required — metadata + instructions
├── references/       # Optional — documentation loaded on demand
├── scripts/          # Optional — executable code
└── assets/           # Optional — templates, schemas, static resources
```

`SKILL.md` has required YAML frontmatter and Markdown body:

```markdown
---
name: my-skill
description: What this skill does and when to use it. Include specific keywords.
license: MIT
metadata:
  author: yourname
  version: "1.0"
---

# My Skill

Instructions the agent follows when this skill is activated.
```

Two required fields. `name` must be lowercase with hyphens, max 64 characters. `description` drives when the skill activates — agents match it against your request, so specific keywords matter more than clever prose.

The body is unconstrained Markdown. Write whatever helps the agent do the task. Step-by-step workflows, decision trees, output format templates, links to reference files — all valid.

The design principle behind the format is **progressive disclosure**. At startup, only the `name` and `description` load — roughly 100 tokens per skill, so you can have dozens without bloating context. When a task matches a skill's description, the full `SKILL.md` body loads. Reference files in `references/` only load when the agent explicitly reaches for them. This keeps context focused regardless of how many skills are installed.

---

## What `npx skills` Does

At its core, `npx skills add` takes a source, finds the skills in it, and installs them to the right location for each detected agent.

```bash
npx skills add huyhoang1001/dev-muscle-memory
```

That command:
1. Clones the repository (or fetches via API — it's fast)
2. Scans for `SKILL.md` files in standard discovery paths
3. Detects which coding agents you have installed by checking known binary paths and config locations
4. Asks which skills you want (or takes flags to skip the prompt)
5. Offers symlink or copy installation
6. Drops files in the right place for each agent

**Symlink installation** is the default and the right choice for most cases. A single canonical copy lives in a shared location; each agent directory gets a symlink pointing to it. Updates to the source flow to all agents immediately. You run `npx skills update` once instead of once per agent.

**Copy installation** makes independent copies per agent. Use this when symlinks aren't supported (Windows without developer mode, some network filesystems) or when you want agents to diverge — edit one agent's version without affecting others.

### The full command surface

```bash
# Add a skill from GitHub (shorthand, full URL, direct skill URL, SSH, local path)
npx skills add owner/repo
npx skills add https://github.com/owner/repo
npx skills add https://github.com/owner/repo/tree/main/skills/my-skill
npx skills add git@github.com:owner/repo.git
npx skills add ./my-local-skills

# Use a skill without installing (pipe to your agent)
npx skills use owner/repo@skill-name | claude
npx skills use owner/repo --skill my-skill --agent claude-code

# List installed skills
npx skills list
npx skills ls -g                    # global only
npx skills ls -a kiro-cli cursor    # filter by agent

# Search for skills
npx skills find
npx skills find rust                # keyword search
npx skills find react --owner vercel  # search within an org

# Update
npx skills update                   # all skills
npx skills update my-skill          # specific skill
npx skills update -g                # global only

# Remove
npx skills remove my-skill
npx skills remove --all -y          # everything, no prompt

# Create a new skill scaffold
npx skills init my-new-skill
```

### Useful flags for real workflows

```bash
# CI/CD friendly — non-interactive, no prompts
npx skills add owner/repo --all -y

# Install only specific skills
npx skills add owner/repo --skill rust-code-review --skill rust-dev

# Install globally — available in every project
npx skills add owner/repo -g

# Install to specific agents only
npx skills add owner/repo -a kiro-cli -a claude-code

# Preview without touching anything
npx skills add owner/repo --list
```

### Private repositories

```bash
# GitHub shorthand or HTTPS — uses Git credential helper, GitHub CLI, then SSH fallback
npx skills add acme/private-skills

# Explicit SSH
npx skills add git@github.com:acme/private-skills.git

# Token via environment variable (optional, for CI)
GITHUB_TOKEN=ghp_xxx npx skills add acme/private-skills
```

---

## How Skills Are Discovered in a Repository

When you run `npx skills add`, the CLI walks known paths looking for `SKILL.md` files. You don't need a manifest. You don't need to register anything. The CLI looks in:

- Root directory (if it contains `SKILL.md`)
- `skills/` — the canonical location
- `.kiro/skills/`, `.claude/skills/`, `.cursor/skills/`, and every other agent path

This means a repository can mix skills targeting different audiences in different folders, and the CLI will find all of them. The `skills/` folder is the convention for repositories that exist *to publish skills*. Agent-specific paths are for skills that live alongside a project.

Walk depth is bounded to three levels by default — `skills/<name>/SKILL.md`, `skills/<category>/<name>/SKILL.md`, and `skills/<category>/<subcategory>/<name>/SKILL.md`. Use `--full-depth` if you have skills nested deeper.

---

## The 72 Agent Problem (And Why It's Actually Solved)

The agent landscape is fragmented by design. Different tools, different companies, different philosophies about where configuration lives. Here's a sample of what the path matrix looks like:

| Agent | Project path | Global path |
|-------|-------------|-------------|
| Kiro CLI | `.kiro/skills/` | `~/.kiro/skills/` |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Cursor | `.agents/skills/` | `~/.cursor/skills/` |
| Codex | `.agents/skills/` | `~/.codex/skills/` |
| Windsurf | `.windsurf/skills/` | `~/.codeium/windsurf/skills/` |
| GitHub Copilot | `.agents/skills/` | `~/.copilot/skills/` |
| Goose | `.goose/skills/` | `~/.config/goose/skills/` |
| Gemini CLI | `.agents/skills/` | `~/.gemini/skills/` |

That's not even a tenth of the 72 supported agents. `npx skills` knows all 72 paths, detects which binaries are present on your system, and handles the routing. You write once. Everyone installs with the same command.

This is the npm moment for agent context. Before npm, distributing JavaScript libraries meant zip files, manual copying, careful README instructions, hoping people read them. After npm, `npm install`. Before `npx skills`, distributing agent context meant per-tool installation instructions that immediately went stale. After `npx skills`, one command.

---

## Publishing Your Own Skills

Making your skills installable via `npx skills` requires nothing beyond putting `SKILL.md` files in the right place in a public GitHub repository. To match community conventions (and get the most out of skills.sh), the recommended structure is:

```
your-skills-repo/
├── README.md
├── AGENTS.md              # guidance for agents contributing to this repo
├── skills.sh.json         # grouping metadata for the skills.sh directory
└── skills/
    ├── my-first-skill/
    │   ├── SKILL.md
    │   ├── metadata.json  # version, author, abstract, references
    │   └── references/
    │       └── detailed-guide.md
    └── my-second-skill/
        ├── SKILL.md
        └── metadata.json
```

**`skills.sh.json`** lets you define groupings that appear on your skills.sh profile page:

```json
{
  "$schema": "https://skills.sh/schemas/skills.sh.schema.json",
  "groupings": [
    {
      "title": "Rust",
      "skills": ["rust-code-review", "rust-dev"]
    }
  ]
}
```

**`metadata.json`** per skill gives the directory rich context:

```json
{
  "version": "1.0.0",
  "author": "yourname",
  "date": "August 2026",
  "abstract": "What this skill does in detail.",
  "references": ["https://relevant-docs.example.com"]
}
```

```bash
# Anyone can then install with:
npx skills add yourusername/your-skills-repo
```

That's it. No npm publish. No registry submission. No approval process. Public GitHub repository with valid `SKILL.md` files — discoverable and installable immediately.

For discoverability, submit to [skills.sh](https://skills.sh/), the community directory and leaderboard. As of mid-2026 it tracks 69,000+ skills and 2 million CLI installs. The leaderboard surfaces what people are actually using, which creates a flywheel: popular skills get installed more, appear higher in search, get installed more.

---

## Security: The Part That Matters

Fast growth creates attack surface. The skills ecosystem learned this early.

The critical insight from the security research Socket presented at Skills Night: a skill can look completely clean at the Markdown level but include a Python file in `scripts/` that opens a remote shell. You would never catch that by reading the `SKILL.md`. You have to audit every file.

The response from the skills ecosystem was to build security infrastructure rather than slow down distribution:

**Socket** runs cross-ecosystem static analysis combined with LLM-based noise reduction on submitted skills — 95% precision, 98% recall.

**Gen** is building Sage, a real-time agent trust layer that monitors every connection in and out of your agents, catching data exfiltration and prompt injection at runtime rather than at install time.

**Snyk** brings their package security background to the skills context.

The practical implication: treat skills from community authors the same way you treat npm packages. Install from authors you trust, read the contents before installing anything from an unknown source, prefer skills that have been audited on skills.sh. `npx skills` itself reminds you of this at install time: "Review skills before use; they run with full agent permissions."

For team or enterprise use, private repositories work natively — the CLI respects your existing Git credential configuration, GitHub CLI authentication, or SSH keys without requiring special setup.

---

## What Makes a Good Skill (For Publishing)

The description field is everything. It drives both automated skill activation and human discovery on skills.sh. Spend time on it.

**Bad:**
```
description: Helps with Rust code.
```

**Good:**
```
description: Senior-level Rust code review covering unsafe soundness,
async/cancellation hazards, ownership patterns, error handling, and performance.
Use when reviewing Rust git changes or preparing code for merge.
```

The difference: the good description includes the specific concerns the skill addresses, the kinds of code it's relevant to, and when to invoke it. Agents use this for activation matching. Humans use it to decide whether to install.

Keep `SKILL.md` under 500 lines. Move detailed reference material — checklists, pattern catalogs, decision trees — into `references/`. Agents load references on demand, so detailed is fine as long as it's separated. The main skill body should be actionable on its own.

Tell the agent what NOT to do. Explicit negative instructions ("do not implement fixes until the user confirms") are as important as positive instructions. Without them, agents default to their training, which is often to be maximally helpful in ways that don't match your workflow.

---

## The Ecosystem in 2026

The numbers tell the story: 72 supported agents, 29k stars on the CLI repository, 69k skills tracked on skills.sh, 2 million installs. In the seven months since launch.

A few things from Skills Night in San Francisco that signal where this is going:

**Documentation sites are auto-generating skills.** Mintlify now generates a skill for every documentation site it hosts — Claude Code's own docs, Coinbase, Perplexity, Lovable. Traffic to these sites is 50% coding agents, up from 10% a year ago. The skill is no longer something a docs team writes; it's a byproduct of having well-structured documentation.

**Skills are becoming a CI layer.** Sentry's David Cramer built Warden, which runs skills as linters on pull requests via GitHub Actions. Agents become a static analysis layer, not just an interactive assistant.

**Native development is within scope.** Evan Bacon from Expo showed native iOS feature upgrades driven entirely by Claude Code using Expo skills — SwiftUI components, gesture transitions, automatic crash fixes in their production app Expo Go. The LLDB integration skill lets agents read the native iOS view hierarchy. Skills are how the agent knows enough about your specific stack to be useful at this level.

The trajectory is clear: skills are becoming the mechanism by which specialized knowledge is distributed to agents at scale, the same way npm packages are the mechanism by which reusable code is distributed to applications.

---

## Quick Reference

```bash
# Install skills from a GitHub repo
npx skills add owner/repo

# Preview what's available
npx skills add owner/repo --list

# Install specific skills to specific agents
npx skills add owner/repo --skill my-skill -a kiro-cli -a claude-code

# Kiro users: install directly into .kiro/skills/ (Kiro isn't a CLI binary so needs --copy)
npx skills add owner/repo --all -a kiro-cli --copy -y

# Install globally
npx skills add owner/repo -g

# Check for updates
npx skills update

# Find skills by keyword
npx skills find rust

# Use a skill without installing
npx skills use owner/repo@skill-name | claude

# See what's installed
npx skills list
```

**Key links:**
- CLI: [github.com/vercel-labs/skills](https://github.com/vercel-labs/skills)
- Directory: [skills.sh](https://skills.sh/)
- Specification: [agentskills.io/specification](https://agentskills.io/specification)
- `dev-muscle-memory`: [github.com/huyhoang1001/dev-muscle-memory](https://github.com/huyhoang1001/dev-muscle-memory)

---

*Content was paraphrased for compliance with licensing restrictions. Sources: [vercel.com/blog/introducing-skills](https://vercel.com/blog/introducing-skills), [vercel.com/blog/skills-night-69000-ways-agents-are-getting-smarter](https://vercel.com/blog/skills-night-69000-ways-agents-are-getting-smarter), [github.com/vercel-labs/skills](https://github.com/vercel-labs/skills), [agentskills.io/specification](https://agentskills.io/specification)*
