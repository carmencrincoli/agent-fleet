---
description: "Plan validation, AI-DLC guardrail, plan creation and management"
applyTo: "**"
---
# Plan — AI Validation Guardrail

## Core Rule
AI output is **untrusted until validated**. Every task follows:

**Plan → Generate → Verify → Iterate**

Never: Plan → Generate → Done

## Requirements
1. Every plan MUST include an AI validation section before implementation
2. Plans without a validation strategy are **incomplete**
3. AI generation tasks must always be followed by verification tasks
4. Plans live in `Plans/` as numbered files (e.g., `Plans/001-plan-feature.md`)

## Workflow
- "create a plan for [X]" → AI creates numbered plan with tasks + validation
- "work on plan [ID]" → AI executes tasks, verifying each one

> Skill: See `plan-management` skill for full plan creation workflow
