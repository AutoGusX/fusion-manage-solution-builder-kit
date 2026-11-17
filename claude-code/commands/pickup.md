---
description: Load project context and summarize where you left off + what's next
argument-hint: "(optional focus, e.g. a spec number or feature area)"
---

Orient the user in the **current project** so they can resume work fast. This is a
**read-only** briefing—do NOT edit anything or start implementing.

Optional focus: **$ARGUMENTS**

Gather context, then summarize:

1. **Read `CLAUDE.md`** in the project root (if present) for architecture, conventions, and commands.
2. **Read `BACKLOG.md`** (if present)—note the items under `Now` and `Next`.
3. **Scan `specs/`**—list specs and read any with `Status: in-progress` (or the highest-numbered / most recently modified if the focus argument points at one).
4. **Check version** if this is an extension (`manifest.json` / `package.json`) and report the current version.
5. **Check git state** if the project is a repo (`git status`, recent `git log`) to see uncommitted work or the last thing done.

Then give a tight briefing:
- **What this project is** (one line).
- **Where things stand**—current version, in-progress spec, uncommitted changes.
- **Top of the backlog**—the next 1-3 items, with their spec links.
- **Recommended next action**—the single most sensible thing to do next, and offer to start it (via plan mode if it's a spec'd feature).

Keep it scannable. The goal is: read this, know exactly what to do next.
