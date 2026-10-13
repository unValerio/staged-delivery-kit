# Staged delivery kit

Portable Claude Code skill + three subagents for implementing a software
development plan in self-validated blocks, closed by an independent
validator + reviewer gate (normally one, at completion).

## Contents

```
.claude/skills/staged-delivery/SKILL.md   # coordinator workflow (main session)
.claude/agents/staged-implementer.md      # writes code, TDD + real self-validation
.claude/agents/staged-validator.md        # independent QA via real entry points
.claude/agents/staged-reviewer.md         # read-only cumulative code review
```

## Models

| Role | Model | Effort |
| --- | --- | --- |
| Coordinator (your session) | any — Opus 5.5 medium/high recommended | — |
| `staged-implementer` | `claude-sonnet-5-5` | high |
| `staged-validator` | `claude-sonnet-5-5` | high |
| `staged-reviewer` | `claude-opus-5-5` | medium |

Pinned via the `model` and `effort` frontmatter keys. To change one, edit the
agent file. The `CLAUDE_CODE_EFFORT_LEVEL` env var overrides the frontmatter
`effort`, so leave it unset if you want the per-role levels.

## Install in another project

```bash
cp -r staged-delivery-kit/.claude <target-repo>/
# or user-wide: copy skills/ and agents/ into ~/.claude/
```

Make sure the target repo gitignores `tmp/` (plans and evidence go to
`<worktree>/tmp/<task>/`).

Check: start `claude` in the target repo, run `/agents` (the three roles appear
with their models), and confirm `/staged-delivery` is listed as a skill.

## Usage

```
/staged-delivery Implement docs/plan.md. Commits allowed; no push/PR.
```

The coordinator writes or adopts `tmp/<task>/PLAN.md`, splits it into blocks
with owned paths, runs implementers (in parallel when paths do not overlap),
then freezes the source and launches validator + reviewer together at the gate,
routing accepted findings back to the implementer until both pass.

Authority is explicit: using the skill does not authorize push, PR, deploy or
merge — say so in the request.

## Credits

The implementer's minimality ladder is adapted from
[Ponytail](https://github.com/DietrichGebert/ponytail).

## License

[MIT](LICENSE)
