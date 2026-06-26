---
name: fleet-loop
description: "Autonomous fleet loop: Planner → Researcher → Critic → Execute → Validator. Repeats until all tasks pass. Triggers: fleet, fleet loop, run fleet, autonomous loop."
---

# 🔁 Fleet Loop — Autonomous Agent Orchestration

**IMMEDIATELY begin the fleet loop. Do NOT wait for additional input.**

## How It Works
This skill runs an autonomous loop with four specialized read-only agents plus you (the primary, who
implements). Sub-agents are stateless and cannot write files or run SQL — **only you** persist artifacts and
pass context into each delegation.

1. **Plan** — @planner creates a plan with tasks and acceptance criteria
2. **Research** — @researcher gathers context and emits a digest JSON
3. **Digest** — *you* validate the digest JSON and write it to `plans/digests/<plan-id>.json`
4. **Critique** — @critic reviews the plan against the digest (pre-build gate; runs only when triggered)
5. **Execute** — *you* implement each task (reading the digest first)
6. **Validate** — @validator checks every task against its criteria (CEARFS)
7. **Loop** — on critic `blocking`, revise the plan; on validator FAIL, fix and re-validate

## Canonical Loop Ordering
The critic runs **after** research/digest and **before** execution — never before the digest exists.

```
MAX_VALIDATE_RETRIES = 3
MAX_CRITIC_CYCLES = 2

Phase 1: plan = @planner(mission)
Phase 2: research = @researcher(plan)
Phase 3: digest = validate_and_write(research.json)        # you; advisory artifact
Phase 4: if critic_triggered(plan):
             cycle = 0
             loop:
                 verdict = @critic(plan, digest)
                 if verdict != "blocking" or cycle >= MAX_CRITIC_CYCLES: break
                 plan = @planner(mission, plan, critic.findings)   # revise
                 if scope_or_files_changed(plan):
                     research = @researcher(plan); digest = validate_and_write(research.json)
                 cycle++
Phase 5: implement each task (read digest first)
Phase 6: result = @validator(plan)                          # reads digest first
         if FAIL: fix and re-validate (≤ MAX_VALIDATE_RETRIES)
         if ALL PASS: coverage check (if applicable) → commit → report
```

## Phase 1: Planning
Hand the mission to @planner. Receive a structured plan with numbered tasks, acceptance criteria, and
validation commands. If the plan seems incomplete, ask @planner to refine before proceeding.

## Phase 2: Research
Ask @researcher to find relevant files/patterns, conventions to follow, and risks. The researcher returns a
markdown report **and** a single fenced ```json digest block.

## Phase 3: Digest — persist (CANONICAL definition lives here)
**You** (the primary) own the digest; the read-only sub-agents cannot write it.

### Canonical path
`plans/digests/<plan-id>.json` (committed; `<plan-id>` matches the plan's number, e.g. `071`).

### Canonical schema (single source of truth — all agents reference this, never redefine it)
```json
{
  "schema_version": 1,
  "plan_id": "string",
  "generated_at": "ISO-8601 string",
  "source_plan_path": "plans/NNN-*.md",
  "git_head": "short sha (best-effort, may be omitted)",
  "research_scope": "one-line description",
  "relevant_files": [{ "path": "string", "purpose": "string", "confidence": 0.0 }],
  "patterns": ["string"],
  "gotchas": ["string"],
  "reuse_notes": [{ "path": "string", "trust": "high | low" }]
}
```

### Write procedure
1. Extract the single ```json block from the researcher's output.
2. Validate it: it must parse (e.g. PowerShell `ConvertFrom-Json`) **and** contain the required top-level
   keys (`schema_version`, `plan_id`, `relevant_files`).
3. On success → write/overwrite `plans/digests/<plan-id>.json`.
4. On invalid JSON → re-run @researcher **once**; if still invalid, write **no** digest and proceed
   search-based (the digest is advisory, never required).

### Conditional creation (trigger)
Create a digest only when the plan is non-trivial — same signal set as the critic trigger below
(multi-session / high-context / behavior-or-state-changing). Skip it for trivial single-session plans.

### Read contract (three cases)
- **New plan**: no digest exists yet — @planner plans from the repo instructions + mission only.
- **After research**: @critic, you (implementer), and @validator read the digest first.
- **Resumed/revised plan**: @planner MAY read an existing digest first, but MUST treat it as stale unless
  refreshed.

All readers: **read the digest first to avoid redundant discovery, but still search** when confidence/trust
is low, the plan changed, files are missing, or the digest does not cover the task (advisory, not exclusive).

### Lifecycle / staleness
A digest is **stale** when the plan materially changed, relevant files changed, the digest predates the
working branch, or the critic/validator finds a mismatch — refresh it. When the plan is **completed,
abandoned, or superseded**, archive or delete its digest (e.g. remove `plans/digests/<plan-id>.json` when
the plan is moved to `plans/archive/`).

## Phase 4: Critique (pre-build gate)
Run @critic only when the **trigger policy** fires. Run the critic when ANY are true:
- plan has > 3 tasks
- changes agent / skill / instruction behavior
- introduces persistence or state
- adopts external-framework ideas
- changes orchestration flow
- has unclear acceptance criteria
- spans multiple sessions

Otherwise you MAY skip the critic with an explicit one-line note ("Critic skipped: trivial single-session
plan").

Pass @critic the plan content + the digest path/content. It returns severity-tagged findings and a verdict
`pass | warning | blocking`.

### On `blocking`
Loop back to @planner with: **original mission + current plan + critic blocking findings + digest path**.
@planner returns a revised plan. If the revision changes research scope or touched files, re-run targeted
@researcher and refresh the digest before continuing. **Bound: max 2 critic revision cycles**, then stop and
surface the impasse to the user.

### On `warning` / `pass`
Continue to execution; record any warnings.

## Phase 5: Execution
Implement each task in order. **Read `plans/digests/<plan-id>.json` first**, honoring `reuse_notes.trust`
(low → re-verify before relying) and `confidence` (low → prioritize scrutiny). Follow patterns the digest
and @researcher identified. After each task, do a quick self-check. Search when the digest is thin or stale.

## Phase 6: Validation
Hand the plan and implementation to @validator (it reads the digest first):
- @validator checks every task against its acceptance criteria and scores the CEARFS rubric.
- Returns PASS/FAIL per task with evidence.

### On FAIL
1. Read the failure evidence carefully.
2. Fix the specific issues @validator identified.
3. Re-run @validator on the failed tasks only.
4. If still failing after 3 retries, stop and report what was attempted, what keeps failing, and suggested
   next steps.

### On ALL PASS
1. **Test coverage check** — applies when the plan modified any of `fleet-config` `SOURCE_DIRS`:
   - Run `fleet-config` `TEST_COMMAND` — zero failures required.
   - Run `fleet-config` `COVERAGE_COMMAND` — must not regress `COVERAGE_FLOOR`.
   - If `TEST_COMMAND` is `N/A`, skip this gate. If `LINT_COMMAND` is set, run it too.
   - If coverage dropped: add tests before proceeding (treat as a FAIL, loop back to Phase 5).
2. Commit changes: `git add -A && git commit -m "Complete: [plan title]"`
3. Push if a remote exists: `git push`
4. Output the completion report.

## Completion Report
```
✅ FLEET COMPLETE
Plan: [title]
Tasks: [X/X passed]
Critic cycles: [N]   Validate retries: [N]
Summary: [one-line description of what was done]
```
