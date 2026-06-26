---
description: "Quality rubric detail, CEARFS scoring guide, dimension definitions, 1-5 scale, validation gate, scoring threshold, rework trigger, plan validation, work output validation, rubric examples, quality dimensions"
---
# Quality Rubric — Full Scoring Guide

## Dimensions & 5-Level Scales

### C — Correctness
| Score | Description |
|-------|-------------|
| 1 | Fundamentally broken — wrong output, crashes, logic errors |
| 2 | Partially works but has significant bugs or incorrect behavior |
| 3 | Core logic correct — minor edge-case bugs only |
| 4 | Correct in all tested scenarios, handles known edge cases |
| 5 | Provably correct — comprehensive coverage, no known issues |

### E — Completeness
| Score | Description |
|-------|-------------|
| 1 | Major requirements missing — less than half addressed |
| 2 | Some requirements met but significant gaps remain |
| 3 | All core requirements met — only minor items missing |
| 4 | All requirements met including edge cases and docs |
| 5 | Exceeds requirements — anticipates follow-up needs |

### A — Adherence
| Score | Description |
|-------|-------------|
| 1 | Ignores conventions — inconsistent with codebase patterns |
| 2 | Partially follows conventions — noticeable deviations |
| 3 | Follows main conventions — minor style inconsistencies |
| 4 | Consistent with all project patterns and instructions |
| 5 | Exemplary adherence — could be used as a reference |

### R — Robustness
| Score | Description |
|-------|-------------|
| 1 | No error handling, no tests — fails on any unexpected input |
| 2 | Handles happy path only — crashes on edge cases, no test coverage |
| 3 | Handles common errors — test files exist for new/changed modules, `fleet-config` `TEST_COMMAND` passes |
| 4 | Comprehensive error handling — tests cover happy path + error paths, coverage does not regress |
| 5 | Battle-hardened — adversarial tests, contract tests for cross-module coupling, coverage improves |

> **Test requirement**: R ≥ 3 requires that new or modified source modules in the project's `SOURCE_DIRS`
> (see `fleet-config.instructions.md`) have corresponding test files, and the `fleet-config` `TEST_COMMAND`
> passes. R < 3 is automatic if code ships without tests. If `TEST_COMMAND` is `N/A` (no test toolchain),
> judge R by inspection (error handling, edge cases) instead.

### F — Formatting
| Score | Description |
|-------|-------------|
| 1 | Unreadable — inconsistent style, poor naming, no structure |
| 2 | Messy — some structure but hard to follow |
| 3 | Clean — readable, consistent style, reasonable naming |
| 4 | Well-organized — clear structure, good naming, easy to navigate |
| 5 | Exemplary — publication-quality, self-documenting |

### S — Security
| Score | Description |
|-------|-------------|
| 1 | Critical vulnerabilities — injection, auth bypass, data exposure |
| 2 | Significant issues — improper validation, weak auth |
| 3 | No obvious vulnerabilities — basic security practices followed |
| 4 | Secure by design — input validation, least privilege, safe defaults |
| 5 | Defense in depth — multiple layers, audit logging, threat-modeled |

## Scoring Rules
1. **Every dimension must score ≥3.** A single dimension <3 triggers mandatory rework.
2. The average score is informational — it does NOT override the per-dimension gate.
3. Score N/A with explanation if a dimension genuinely doesn't apply (e.g., Security for a docs-only change).
4. Round the average to one decimal place.

## Validation Contexts

### Plan Validation
When validating a **plan** (not implementation), score dimensions as:
- **C**: Is the plan logically sound? Will it achieve the objective?
- **E**: Are all requirements captured? Dependencies mapped?
- **A**: Does it follow the plan template and AI-DLC guardrails?
- **R**: Does it handle failure modes? Include rollback steps?
- **F**: Is the plan well-structured and easy to follow?
- **S**: Are security considerations addressed where relevant?

### Work Output Validation
When validating **completed work**, score dimensions against the implementation:
- **C**: Does the code/config work correctly?
- **E**: Are all acceptance criteria met?
- **A**: Does it follow project conventions and patterns?
- **R**: Are edge cases and errors handled?
- **F**: Is the output clean, readable, well-named?
- **S**: Are there any security issues?

## Cheatsheet
```
Rubric: C=X E=X A=X R=X F=X S=X → avg X.X → PASS/FAIL

PASS: All dimensions ≥3
FAIL: Any dimension <3 → list failing dimensions → rework required

Dimensions:
  C = Correctness    (right behavior, no bugs)
  E = Completeness   (all requirements met)
  A = Adherence      (follows conventions)
  R = Robustness     (handles errors/edge cases)
  F = Formatting     (clean, readable, well-structured)
  S = Security       (no vulnerabilities, safe defaults)
```

## Full Example — PASS
```
## Quality Rubric
| Dim | Score | Evidence |
|-----|-------|----------|
| C | 4 | All functions return correct output, edge cases handled |
| E | 4 | All 5 acceptance criteria met, docs updated |
| A | 3 | Follows naming convention, one minor style deviation in helper |
| R | 3 | Error handling present, timeout case not covered but unlikely |
| F | 4 | Clean structure, consistent formatting, clear variable names |
| S | 3 | Input validation present, no auth issues, uses parameterized queries |

Rubric: C=4 E=4 A=3 R=3 F=4 S=3 → avg 3.5 → PASS
```

## Full Example — FAIL
```
## Quality Rubric
| Dim | Score | Evidence |
|-----|-------|----------|
| C | 4 | Logic correct, tests pass |
| E | 2 | ❌ Missing requirement #3 (error logging), no docs |
| A | 3 | Follows patterns |
| R | 1 | ❌ No error handling — crashes on empty input |
| F | 4 | Clean formatting |
| S | 3 | No issues |

Rubric: C=4 E=2 A=3 R=1 F=4 S=3 → avg 2.8 → FAIL
Failing dimensions: E (2), R (1)
Rework required: Add error logging (requirement #3), add error handling for empty input
```
