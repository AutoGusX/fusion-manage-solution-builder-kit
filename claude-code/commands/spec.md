---
description: Scaffold a new numbered feature spec + backlog entry, then offer to plan it
argument-hint: <short feature idea>
---

You are creating a new feature spec for the **current project** (the cwd you're running in). The user's idea:

**$ARGUMENTS**

Work through these steps in order:

1. **Locate structure.** Look for a `specs/` directory in the project root—create it if missing. Check whether `BACKLOG.md` exists in the root.

2. **Determine the next spec number.** List files in `specs/` matching `NNNN-*.md`, take the highest 4-digit prefix, and increment it (zero-padded, e.g. `0007`). If none exist, start at `0001`.

3. **Interview before writing.** Ask 2-3 focused questions to pin down anything ambiguous about the problem, scope, or acceptance criteria—only what you genuinely need. If the idea is already crisp, skip straight to drafting and just confirm your understanding in one line.

4. **Draft the spec.** Create `specs/NNNN-<kebab-slug>.md` using the section structure from the template at `~/.claude/templates/spec-template.md` (Problem, Goals/Non-goals, Proposed approach, Acceptance criteria, Risks/open questions, Out of scope). Fill in every section from the idea plus the interview answers. Set `Status: draft`, `ID` to the number, and `Date` to today. Acceptance criteria MUST be concrete and checkable—`/verify` and `/code-review` will use them later.

5. **Update the backlog.** Add a checkbox item under `Now` (or `Next` if the user says it's not immediate) in `BACKLOG.md`, formatted:
   `- [ ] <feature> (spec: specs/NNNN-<slug>.md)`
   Create `BACKLOG.md` from `~/.claude/templates/BACKLOG-template.md` first if it doesn't exist.

6. **Hand off.** Show the spec path and a one-line summary, then offer to enter **plan mode** to turn the approved spec into an implementation plan. Do NOT start implementing code.
