---
name: plan-management
description: "Plan creation with AI-DLC validation guardrails. Triggers: plan, create plan, new plan, work on plan, AI validation, verification strategy."
---

# plan-management — AI Validation Guardrail (AI-DLC)

## Purpose
Creates and manages plans with built-in AI validation guardrails. Enforces the Plan → Generate → Verify → Iterate lifecycle.

## Triggers
- "plan" / "create plan" / "new plan"
- "work on plan [ID]"
- "AI validation" / "verification strategy"

## Core Rule
**AI output is untrusted until validated.**

Every plan involving AI-assisted work MUST include an AI validation section before implementation tasks. Plans without a validation strategy are incomplete.

## Plan Creation Workflow

### 1. Create Plan File
- Location: `Plans/NNN-plan-[short-name].md`
- Number sequentially: 001, 002, 003...
- Include: Tasks, acceptance criteria, AI validation section

### 2. Plan Template
```markdown
# Plan NNN — [Title]

## Objective
[What this plan accomplishes]

## AI Usage Declaration
- **AI will generate**: [list what AI produces]
- **AI will NOT be trusted to decide**: [list human decisions]
- **Expected failure modes**: hallucinated APIs, incorrect logic, missing edge cases, security mistakes

## Validation Strategy
- [ ] [Deterministic test or check for each AI-generated artifact]
- [ ] [Schema validation, compile/build checks]
- [ ] [Regression tests]
- [ ] Quality rubric gate: @validator scores all 6 C·E·A·R·F·S dimensions (each must be ≥3)

## Tasks
- [ ] Task 1: [description]
  - Verify: [how to confirm this is correct]
- [ ] Task 2: [description]
  - Verify: [how to confirm this is correct]

## Status
- Created: [date]
- Status: Draft / In Progress / Complete / Archived
```

### 3. Execution Workflow
When user says "work on plan [ID]":
1. Read the plan file
2. Find the next incomplete task
3. Execute the task
4. Run the verification step
5. Mark task complete
6. Move to next task
7. When all tasks done, run final validation (includes C·E·A·R·F·S quality rubric scoring — all dimensions must be ≥3)

### Rules
- NEVER skip verification steps
- NEVER mark a task complete without running its verify step
- If verification fails, iterate — don't move on
- Archive completed plans (move to `Plans/archive/`)

### Mandatory Test Rule
Any plan that **creates or modifies** files in the project's `SOURCE_DIRS` (see
`fleet-config.instructions.md`) MUST include:
1. At least one task that adds or updates **test files** in `TEST_DIRS`
2. A validation step that runs `fleet-config` `TEST_COMMAND` with zero failures
3. A coverage check: `fleet-config` `COVERAGE_COMMAND` must not regress

If `TEST_COMMAND` is `N/A` (project has no automated tests), this rule is waived.
Plans that only change instructions, skills, or documentation are exempt.

## Pattern
✅ Plan → Review → Execute → Verify → Iterate
❌ Plan → Generate → Done
