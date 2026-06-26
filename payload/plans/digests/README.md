# plans/digests/

Plan-scoped **research digests** written by the fleet loop. One file per plan:
`plans/digests/<plan-id>.json` (e.g. `071.json`).

- **Producer**: `@researcher` emits the digest JSON; the **primary** validates and writes it here.
- **Consumers**: `@critic`, the implementer, and `@validator` read it first to avoid redundant discovery.
- **Advisory, not authoritative**: read first, but still search when confidence/trust is low, the plan
  changed, files are missing, or the digest is stale.
- **Canonical schema + lifecycle**: defined once in `.github/skills/fleet-loop/SKILL.md`. Digests are
  created conditionally (non-trivial / multi-session plans) and removed/archived when their plan is
  completed, abandoned, or superseded.

Do not hand-edit these files; regenerate via the fleet loop.
