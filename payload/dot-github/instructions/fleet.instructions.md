---
description: "Fleet agents and autonomous loop. Planner, Researcher, Critic, Validator agents with fleet-loop skill."
applyTo: "**"
---
# Fleet — Multi-Agent Loop

This workspace has a fleet of 4 read-only agents orchestrated by the `fleet-loop` skill.

## Agents
| Agent | Role | Rules |
|-------|------|-------|
| @planner | Creates plans with acceptance criteria | Plan only, never implement |
| @researcher | Searches code/docs for context; emits a digest JSON | Research only, never modify |
| @critic | Critiques the PLAN — assumptions, scope, ordering, deps, over-engineering | Critique only, never implement |
| @validator | Checks finished work against criteria (CEARFS PASS/FAIL) | Validate only, never fix |

**Critic vs validator**: @critic is a **pre-build** gate (is the plan right + simple?), runs after research
and only when the trigger policy fires (see `fleet-loop`); @validator is the **post-build** CEARFS gate
(was it built right?). They do not overlap — the critic never scores CEARFS or nitpicks style.

## Invoking the Fleet
Say any of: "run fleet", "fleet loop", "autonomous loop"
→ Invokes the `fleet-loop` skill → Plan → Research → Digest → Critique → Execute → Validate loop

## Loop Rules
- Every task must have acceptance criteria
- The research digest (`plans/digests/<plan-id>.json`) is advisory — read first, but still search when stale
- Validation is mandatory — no task is done until @validator says PASS
- On critic FAIL (blocking): revise the plan (max 2 cycles). On validator FAIL: fix and re-validate (max 3)
- On ALL PASS: commit and report
