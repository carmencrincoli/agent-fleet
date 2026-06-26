# SETUP.md — Fleet Installer (read this, then act)

You are an AI agent installing the **agent fleet** into the current project. This file is an executable
runbook: read it fully, then perform the steps. The fleet is a set of read-only Copilot CLI sub-agents
(planner / researcher / critic / validator) plus an orchestration skill that runs
**Plan → Research → Digest → Critique → Execute → Validate**, gated by a CEARFS quality rubric.

Everything you install lives under `payload/` in this distribution. The install is just: copy `payload/`
into the target repo root, then fill in **one** config file for the local stack.

---

## Detect the mode first

- **Fresh install** — the target repo has no `.github/agents/{planner,researcher,critic,validator}.agent.md`.
  Do a full copy.
- **Update / upgrade** — some fleet files already exist. Do a careful merge (see "Updating an existing
  fleet"). **Never blindly overwrite** a customized `fleet-config.instructions.md`.

---

## Step 1 — Copy the payload into the repo root

Copy the contents of `payload/` so they land at the repo root:

```
payload/.github/agents/planner.agent.md          -> .github/agents/planner.agent.md
payload/.github/agents/researcher.agent.md        -> .github/agents/researcher.agent.md
payload/.github/agents/critic.agent.md            -> .github/agents/critic.agent.md
payload/.github/agents/validator.agent.md         -> .github/agents/validator.agent.md
payload/.github/skills/fleet-loop/SKILL.md        -> .github/skills/fleet-loop/SKILL.md
payload/.github/skills/plan-management/SKILL.md    -> .github/skills/plan-management/SKILL.md
payload/.github/instructions/*.instructions.md     -> .github/instructions/
payload/plans/digests/README.md                    -> plans/digests/README.md
```

Create parent directories as needed. Do not copy this `SETUP.md` or the dist `README.md` into the target.

PowerShell one-liner (run from the dist folder):
```powershell
$dest = "C:\path\to\target-repo"
Copy-Item ".\payload\*" $dest -Recurse -Force
```

---

## Step 2 — Configure for the local stack (the ONLY required edit)

Open `.github/instructions/fleet-config.instructions.md` and set its values for THIS project. This is the
single source of truth the fleet reads for paths and commands. Detect the stack and fill it in:

1. **Identify the language / test runner.** Look for `pyproject.toml`/`setup.cfg` (Python → pytest),
   `package.json` (Node → the `test` script), `go.mod` (Go → `go test ./...`), `Cargo.toml` (Rust →
   `cargo test`), etc.
2. **Identify source + test dirs.** Where does shippable code live (`src/`, `app/`, `lib/`, `cmd/`…)?
   Where do tests live (`tests/`, `__tests__/`, `*_test.go`…)?
3. **Set** `SOURCE_DIRS`, `TEST_DIRS`, `TEST_COMMAND`, `COVERAGE_COMMAND`, `COVERAGE_FLOOR`, `LINT_COMMAND`.
4. **No tests?** Set `TEST_COMMAND` and `COVERAGE_COMMAND` to `N/A`. The fleet then skips the test gate and
   the validator judges Robustness by inspection.
5. Leave `PLAN_DIR` / `DIGEST_DIR` as defaults unless the project already has a different plan convention.

Confirm with the human if the stack is ambiguous. **No other file needs editing** — every other fleet file
references `fleet-config` instead of hard-coding paths/commands.

---

## Step 3 — Updating an existing fleet (merge, don't clobber)

If the target already has a fleet:
- **`fleet-config.instructions.md`**: if it already exists, KEEP the existing values; only add any new keys
  introduced by this version. Never overwrite the human's configured paths/commands.
- **Agent files / skills / rubric**: these are the upgrade payload — overwrite them with the new versions,
  but first check `git diff` for any local customizations the human made and preserve them (ask if unsure).
- **`plans/` and `plans/digests/`**: never delete existing plans or digests.
- If the target has extra fleet agents not in this dist (e.g. project-specific specialists), leave them
  untouched.

---

## Step 4 — Verify the install

1. **Frontmatter loads**: each `*.agent.md` has valid YAML frontmatter (`name`, `description`, `tools`);
   each `SKILL.md` has `name` + `description`.
2. **Config resolves**: `fleet-config.instructions.md` has concrete values (no leftover placeholders) and
   the `TEST_COMMAND` actually runs in this repo (or is `N/A`).
3. **Digest parses**: write a tiny sample digest and confirm it parses (e.g. PowerShell `ConvertFrom-Json`)
   with keys `schema_version`, `plan_id`, `relevant_files`.
4. **Smoke test the loop** (optional but recommended): give the fleet a trivial mission ("add a one-line
   note to the README") and confirm Plan → (Research → Digest) → Critique-skip (trivial) → Execute →
   Validate runs end-to-end.

Report what you installed, the `fleet-config` values you set, and anything the human should confirm.

---

## File manifest — what each file does

| File | Role | Edit per project? |
|------|------|-------------------|
| `.github/instructions/fleet-config.instructions.md` | **Single config**: source/test dirs + test/coverage/lint commands | **YES — the only required edit** |
| `.github/instructions/fleet.instructions.md` | Always-loaded fleet roster + loop rules (thin stub) | No |
| `.github/instructions/plan.instructions.md` | Always-loaded AI-DLC guardrail stub (triggers plan-management) | No |
| `.github/instructions/quality-rubric.instructions.md` | CEARFS rubric stub (6 dims, each ≥3) | No |
| `.github/instructions/quality-rubric.detail.instructions.md` | Full CEARFS scoring guide (R dim reads `fleet-config`) | No |
| `.github/agents/planner.agent.md` | Creates the plan (tasks + acceptance criteria) | No |
| `.github/agents/researcher.agent.md` | Gathers context; emits the advisory digest JSON | No |
| `.github/agents/critic.agent.md` | Pre-build plan gate (assumptions/scope/YAGNI); `pass\|warning\|blocking` | No |
| `.github/agents/validator.agent.md` | Post-build CEARFS gate (PASS/FAIL per task) | No |
| `.github/skills/fleet-loop/SKILL.md` | Orchestration loop + canonical digest schema/lifecycle | No |
| `.github/skills/plan-management/SKILL.md` | Plan creation + AI-DLC validation guardrail | No |
| `plans/digests/README.md` | Documents the digest convention | No |

---

## How to use the fleet after install

- **Create a plan**: say "create a plan for X" → plan-management writes `plans/NNN-*.md`.
- **Run the loop**: say "run fleet" / "fleet loop" → runs the full Plan→…→Validate loop.
- **Standalone critique**: hand the critic a plan for a pre-build review.
- The research **digest** is written to `<DIGEST_DIR>/<plan-id>.json`; it's advisory — agents read it first
  but still search when it's thin or stale.
