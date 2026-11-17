# Claude Code setup

Drop these into your own `~/.claude/` (on Windows, `%USERPROFILE%\.claude\`).

| File | Install to | Purpose |
|---|---|---|
| `commands/spec.md` | `~/.claude/commands/` | `/spec <idea>`: interviews you, writes a numbered spec, adds a backlog item, offers to plan |
| `commands/pickup.md` | `~/.claude/commands/` | `/pickup`: read-only briefing on where the project stands and what is next |
| `commands/bump.md` | `~/.claude/commands/` | `/bump`: version bump for MV3 extensions, keeping `manifest.json` and `package.json` in sync |
| `templates/spec-template.md` | `~/.claude/templates/` | Spec structure: problem, goals, approach, acceptance criteria, risks |
| `templates/BACKLOG-template.md` | `~/.claude/templates/` | Now, Next, Later, Done |
| `templates/project-CLAUDE-template.md` | copy into each project as `CLAUDE.md` | Short project memory loaded every session |
| `global-CLAUDE.example.md` | merge into `~/.claude/CLAUDE.md` | Working preferences that apply to every project |

`/spec` and `/pickup` assume the layout described in [`playbook/01-way-of-working.md`](../playbook/01-way-of-working.md): `specs/`, `BACKLOG.md`, `CLAUDE.md`.

## Tips

- Keep `CLAUDE.md` short. It is loaded into every session.
- Put FM API facts in your own gotchas file and point `CLAUDE.md` at it, so the agent reads the traps before it probes a tenant.
- Add a note in `CLAUDE.md` for anything the agent cannot see: how to get a token, which tenant is the sandbox, what is safe to mutate.
