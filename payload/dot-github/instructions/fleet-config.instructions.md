---
description: "Project-local fleet configuration. The fleet-loop skill, plan-management skill, and quality rubric read these values. THIS IS THE ONLY FILE YOU MUST EDIT when installing the fleet in a new project."
applyTo: "**"
---
# Fleet Config — Project-Local Settings (EDIT ME)

This is the **single source of truth** for project-specific paths and commands. The fleet's test/coverage
gate lives here; every other fleet file references these values instead of hard-coding them. Edit this file
to match the local project, and the whole fleet adapts.

> If the project has **no automated tests**, set `TEST_COMMAND` and `COVERAGE_COMMAND` to `N/A`. The fleet
> then skips the test gate, and the validator's Robustness (R) dimension is judged by inspection only.

## Settings

| Key | Value (EDIT) | Meaning |
|-----|--------------|---------|
| `SOURCE_DIRS` | `src/` | Comma-separated dirs whose new/changed code must have tests. Triggers the test gate. |
| `TEST_DIRS` | `tests/` | Where test files live. |
| `TEST_COMMAND` | `pytest tests/ -v` | Run the test suite; zero failures required. Or `N/A`. |
| `COVERAGE_COMMAND` | `pytest --cov --cov-fail-under=60` | Coverage gate that must not regress. Or `N/A`. |
| `COVERAGE_FLOOR` | `60` | Minimum coverage percent (informational). |
| `LINT_COMMAND` | `N/A` | Optional pre-commit lint/format (e.g. `ruff check . && ruff format --check .`). Or `N/A`. |
| `PLAN_DIR` | `plans/` | Where numbered plans live (`plans/NNN-*.md`). |
| `DIGEST_DIR` | `plans/digests/` | Where research digests are written (`<DIGEST_DIR>/<plan-id>.json`). |

## Examples for other stacks
- **Go**: `SOURCE_DIRS=./...`, `TEST_COMMAND=go test ./...`, `COVERAGE_COMMAND=N/A`.
- **Node/TS**: `SOURCE_DIRS=src/`, `TEST_COMMAND=npm test`, `COVERAGE_COMMAND=npm run coverage`, `LINT_COMMAND=npm run lint`.
- **Rust**: `SOURCE_DIRS=src/`, `TEST_COMMAND=cargo test`, `LINT_COMMAND=cargo clippy -- -D warnings`.
- **Docs-only repo**: all of `SOURCE_DIRS`/`TEST_COMMAND`/`COVERAGE_COMMAND` = `N/A`.
