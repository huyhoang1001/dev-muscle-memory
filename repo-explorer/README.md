# Repo Explorer

A skill for AI agents to systematically map and understand any codebase. Produces a structured repo map covering architecture, modules, data flow, dependencies, and conventions.

## Features

- **Top-level scan** - Detect language, build system, entry points, and folder structure
- **Dependency map** - Parse manifests and surface notable or risky dependencies
- **Architecture analysis** - Identify patterns (layered, plugin, event-driven, actor, etc.)
- **Data flow tracing** - Follow the happy path from entry point to output
- **Conventions** - Naming, error handling, testing, docs, and CI/CD patterns
- **Landmine detection** - Flag tricky, undocumented, or fragile areas

## Usage

```
/repo-explorer
```

The skill will explore the current repository and produce a full repo map.

## Workflow

1. **Top-level scan** - Identify project type and tooling
2. **Dependency map** - Parse manifests for libraries and risks
3. **Architecture analysis** - Map modules and detect patterns
4. **Data flow** - Trace the main execution path
5. **Conventions** - Document team norms and idioms
6. **Output** - Structured repo map
7. **Follow-up** - Offer to drill deeper into any area

## Output

```
## Repo Map: [repo name]
- Module overview table
- Architecture pattern description
- Data flow steps
- Dependency table
- Conventions summary
- Key areas and potential landmines
- Suggested starting points for common tasks
```

## Structure

```
repo-explorer/
├── SKILL.md                    # Main skill definition
├── agents/
│   └── agent.yaml              # Agent interface config
└── references/
    ├── patterns-catalog.md     # Architectural patterns and how to spot them
    └── language-hints.md       # Language-specific conventions and idioms
```

## License

MIT
