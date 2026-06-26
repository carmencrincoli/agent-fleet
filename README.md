# fleet-dist — Portable Agent Fleet

A self-contained, project-agnostic distribution of the **planner / researcher / critic / validator** agent
fleet for GitHub Copilot CLI, plus the orchestration loop, plan-management, and CEARFS quality rubric.

It is parameterized: the only project-specific values (source/test dirs, test/coverage/lint commands) live in
**one** file — `fleet-config.instructions.md`. Every other file references that config, so installing into a
new project means *copy + edit one file*.

## Contents
```
fleet-dist/
├── SETUP.md          ← give this to an agent in the target project; it installs/upgrades the fleet
├── README.md         ← this file (human overview)
└── payload/          ← the files that get copied into the target repo root
    ├── .github/
    │   ├── agents/        planner, researcher, critic, validator (.agent.md)
    │   ├── skills/        fleet-loop/, plan-management/ (SKILL.md)
    │   └── instructions/  fleet-config (EDIT), fleet, plan, quality-rubric (+ detail)
    └── plans/digests/  README (digest convention)
```

## Install (two ways)

**Agent-driven (recommended).** In the target project, point an agent at the installer:
> "Read `fleet-dist/SETUP.md` and install the fleet into this project."

The agent copies `payload/` to the repo root, detects the stack, fills in `fleet-config.instructions.md`,
and verifies. It also handles the **upgrade** case (merge, never clobber a customized config).

**Manual.** Copy `payload/*` to the target repo root, then edit
`.github/instructions/fleet-config.instructions.md` for your stack.

## What you must configure
Just `fleet-config.instructions.md`: `SOURCE_DIRS`, `TEST_DIRS`, `TEST_COMMAND`, `COVERAGE_COMMAND`,
`COVERAGE_FLOOR`, `LINT_COMMAND`, `PLAN_DIR`, `DIGEST_DIR`. Set the commands to `N/A` for repos with no test
toolchain. See `SETUP.md` for per-stack examples (Go / Node / Rust / docs-only).

## Requirement
GitHub Copilot CLI (custom `.agent.md` agents + the task tool). The fleet is Copilot-CLI-native.

## Notes
- This `fleet-dist/` folder is a **staging/distribution** artifact — keep it out of the product repo (or
  delete after copying). Don't ship `SETUP.md` / this `README.md` into target projects; only `payload/`.
