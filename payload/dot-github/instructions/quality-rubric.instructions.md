---
description: "Quality rubric scoring gate, CEARFS dimensions, validation quality, scoring threshold"
applyTo: "**"
---
# Quality Rubric — C·E·A·R·F·S Validation Gate

## Dimensions
| Dim | Name | Measures |
|-----|------|----------|
| **C** | Correctness | Functionally right, no bugs or logic errors |
| **E** | Completeness | All requirements addressed, nothing missing |
| **A** | Adherence | Follows conventions, patterns, and instructions |
| **R** | Robustness | Handles edge cases, errors, and bad input |
| **F** | Formatting | Clean structure, naming, style, readability |
| **S** | Security | No vulnerabilities, safe defaults, proper auth |

## Rule
Every dimension must score **≥3** (out of 5). Any dimension <3 triggers rework.

## Quick Format
`Rubric: C=4 E=3 A=4 R=3 F=5 S=4 → avg 3.8 → PASS`

> Detail: See `quality-rubric.detail.instructions.md` for full scoring guide.
