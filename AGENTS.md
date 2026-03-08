# AGENTS.md

## Purpose
This repository is currently a planning and specification workspace for Jarvis.
There is no application source tree yet; the authoritative files today are:
- `docs/architecture_vision.md`
- `SPECIFICATIONS_FINALES.md`
- `README.md` (currently minimal)
Use this file as the operating guide for coding agents working in this repo.

## Current Repository State
- The repo is in a pre-implementation phase.
- No `src/`, `app/`, `tests/`, or equivalent code directories exist yet.
- No verified package manifest is present (`pyproject.toml`, `package.json`, etc.).
- No verified formatter, linter, type-checker, or test runner config exists.
- No Cursor rules were found in `.cursor/rules/`.
- No Copilot instructions were found in `.github/copilot-instructions.md`.

## Source Of Truth
When requirements conflict, use this order:
1. Direct user instructions
2. `AGENTS.md`
3. `SPECIFICATIONS_FINALES.md`
4. `docs/architecture_vision.md`
5. Existing code and config, once they exist

## Product Direction To Preserve
The future implementation direction is already defined in the docs:
- Backend-first architecture
- Python + FastAPI orchestrator
- Local-first execution model
- LiteLLM for routing local vs cloud models
- Ollama as local LLM runtime
- SQLite + `sqlite-vec` for memory
- Open-WebUI as the first client in the MVP
- Strong governance and auditable decision logic

Do not introduce architecture that contradicts those constraints unless the user explicitly asks for it.

## Build / Lint / Test Commands
### Verified commands today
There are currently no verified build, lint, type-check, or test commands in the
repository because no application scaffolding or tool configuration exists yet.
Verified absence:
- no `pyproject.toml`
- no `package.json`
- no `requirements*.txt`
- no `pytest.ini`
- no `tox.ini`
- no `Makefile`
- no `docker-compose.yml` or `compose.yml`

### What agents should do
- Do not invent commands and present them as existing.
- Do not claim linting or tests passed unless the relevant tooling exists.
- If you add a toolchain later, update this section immediately.

### Single-test command
No single-test command is available yet because no test framework is configured.
When a test runner is added in the future, document at least:
- full test suite command
- single-test file command
- single-test case command, if supported
- lint command
- format command
- type-check command
- local dev / run command

## Implementation Guidance For Future Code
Because the repo has no source code yet, the style guidance below is derived from the project specifications rather than from an established codebase.

## Architecture Conventions
- Keep the orchestrator as the policy owner.
- Keep clients thin; business rules belong in the backend.
- Keep routing, memory, and policy concerns separated.
- Prefer explicit decision paths over hidden heuristics.
- Make local/cloud escalation explainable and auditable.
- Design for Open-WebUI first without coupling the backend to one client.

## Code Style Guidelines
### General
- Prefer small, focused modules with one clear responsibility.
- Prefer explicit, readable code over clever compression.
- Favor pure functions for policy and routing decisions when practical.
- Keep side effects near the edges of the system.

### Formatting
- Follow the formatter actually configured in the repo once one exists.
- Until then, use consistent formatting and avoid style churn.
- Keep lines reasonably short for readable diffs.

### Imports
- Group imports by standard library, third-party, then local modules.
- Keep imports explicit; avoid wildcard imports.
- Avoid circular dependencies by keeping module boundaries clean.

### Naming
- Use descriptive names over abbreviations.
- Match the language's standard naming conventions once code exists.
- Prefer names that reflect domain intent: `policy_engine`, `routing_result`, `memory_entry`, `session_context`, `decision_trace`.
- Name booleans as predicates (`is_allowed`, `has_memory`, `should_escalate`).

### Types
- Prefer strong typing everywhere the language/tooling supports it.
- Do not suppress type errors with `as any`, `@ts-ignore`, or similar escapes.
- Model domain entities explicitly instead of passing loose dictionaries around.
- Validate external input at the boundary.

### Error Handling
- Fail clearly and predictably.
- Do not swallow exceptions silently.
- Return actionable errors for policy refusals and unavailable capabilities.
- Preserve enough context in logs to audit routing and policy decisions.

### Configuration And Secrets
- Keep non-sensitive configuration in versioned files.
- Keep secrets in environment variables or a dedicated secret store.
- Never hardcode tokens, keys, or credentials.

### API Design
- Keep the external API simple and stable.
- Prefer explicit request/response models.
- Design streaming events as typed, intentional event families.
- Expose diagnostics that help verify system state without leaking secrets.

### Testing Expectations
- Add tests with new behavior once the codebase exists.
- Prioritize unit tests for policy, routing, and memory logic.
- Add integration tests for orchestrator flows and degraded-mode behavior.
- When fixing bugs, add or update a test that demonstrates the root cause.

### Logging And Auditability
- Prefer structured logs over free-form strings.
- Record key decisions: selected path, triggered rule, fallback, refusal reason.
- Avoid logging sensitive user content unless explicitly required and protected.

## Working In This Repository
- Before scaffolding code, read `SPECIFICATIONS_FINALES.md` and `docs/architecture_vision.md`.
- If you create the first implementation files, also create the missing tool configuration and update this document with verified commands.
- If you introduce a test runner, document how to run one test file and one test case.
- If you add Cursor or Copilot rule files later, merge their instructions into this document rather than duplicating conflicting guidance.

## What Not To Do
- Do not present planned architecture as implemented reality.
- Do not infer non-existent scripts or dev commands.
- Do not add unnecessary infrastructure before the orchestrator skeleton exists.
- Do not move governance logic into the client layer.
- Do not bypass explicit policy boundaries for convenience.

## Required Maintenance
Update this file whenever any of the following becomes true:
- a build command is added
- a lint or format command is added
- a type-check command is added
- a test runner is added
- a single-test invocation becomes available
- source code establishes stronger local conventions
- Cursor or Copilot instructions are added

Until then, treat this repository as a docs-first, specification-only project.
