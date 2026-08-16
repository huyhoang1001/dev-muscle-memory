---
name: repo-explorer
description: "Deep dive into any repository to understand its structure, architecture, key modules, and dependencies. Produces a clear mental map for navigating and working in an unfamiliar codebase."
license: MIT
metadata:
  author: huyhoang1001
  version: "1.0"
---

# Repo Explorer

## Overview

Systematically explore a repository to build a complete understanding of its architecture, entry points, data flow, and conventions. Output a structured map the developer (or another agent) can use to navigate and contribute confidently.

## Workflow

### 1) Top-level scan

- List root files and folders to identify project type and tooling.
- Detect language(s), build system, package manager, and runtime from config files:
  - `Cargo.toml`, `package.json`, `pom.xml`, `pyproject.toml`, `go.mod`, `Makefile`, etc.
- Identify key entry points: `main.rs`, `index.ts`, `app.py`, `main.go`, etc.
- Note special folders: `src/`, `lib/`, `tests/`, `docs/`, `examples/`, `.github/`, etc.

### 2) Dependency map

- Parse the dependency manifest (e.g., `Cargo.toml`, `package.json`) for:
  - Direct vs dev dependencies
  - Notable third-party libraries and their purpose
  - Peer dependencies or optional features
- Flag outdated, unusual, or potentially risky dependencies.

### 3) Architecture analysis

- Identify the main modules/packages and their responsibilities.
- Map relationships between modules (imports, re-exports, trait implementations).
- Detect architectural patterns in use:
  - Layered (presentation → business → data)
  - Plugin/extension system
  - Event-driven / pub-sub
  - Actor model
  - Middleware pipeline
- Identify shared abstractions, core traits/interfaces, and extension points.

### 4) Data flow

- Trace the happy path from entry point to output for the main use case.
- Identify where state lives (globals, singletons, context objects, databases).
- Note async boundaries, concurrency primitives, and synchronization points.
- Highlight critical paths: auth, payments, data writes, network calls.

### 5) Conventions and standards

- Coding style: naming conventions, file organization, module boundaries.
- Error handling strategy: `Result`/`Option`, exceptions, error codes.
- Testing approach: unit vs integration, test helpers, fixtures, mocking patterns.
- Documentation style: inline docs, README, examples folder.
- CI/CD setup: workflows, linting, formatting, test gates.

### 6) Output format

```markdown
## Repo Map: [repo name]

**Language(s)**: ...
**Build system**: ...
**Entry point(s)**: ...

---

## Module Overview

| Module | Responsibility | Key files |
|--------|---------------|-----------|
| ...    | ...           | ...       |

---

## Architecture Pattern

[Description with diagram in text if helpful]

---

## Data Flow (Main Path)

1. ...
2. ...

---

## Dependencies

| Dependency | Purpose | Notes |
|------------|---------|-------|
| ...        | ...     | ...   |

---

## Conventions

- **Naming**: ...
- **Error handling**: ...
- **Testing**: ...
- **Docs**: ...

---

## Key Areas to Know

- **[Module/file]**: [Why it matters]
- ...

---

## Potential Landmines

- [Anything tricky, undocumented, or easy to break]

---

## Suggested Starting Points

For common tasks in this repo:
- **Add a feature**: Start at ...
- **Fix a bug**: Look in ...
- **Write a test**: Follow pattern in ...
```

### 7) Follow-up

After presenting the map, ask:

```markdown
---

## What's next?

I've mapped the repo. Here's what I can do next:

1. **Drill into a module** - Deep dive on a specific area
2. **Trace a data flow** - Follow a specific request/operation end-to-end
3. **Find where to add X** - Locate the right place for a new feature
4. **Explain a specific file** - Walk through any file in detail
5. **Review the codebase** - Hand off to code-review skill

What would you like to explore?
```

## Resources

### references/

| File | Purpose |
|------|---------|
| `patterns-catalog.md` | Common architectural patterns and how to identify them |
| `language-hints.md` | Language-specific conventions and idioms to look for |
