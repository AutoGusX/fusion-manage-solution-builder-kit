# Configuring Fusion Manage as code

An approach for taking a customer from requirements to a configured tenant quickly, with Claude Code doing the repetitive parts and a human approving each step.

## The loop

1. **Requirements.** Capture them as numbered, testable items (for example `BOM-01`, `CHG-03`). Mark assumptions `[A]` so they get confirmed rather than silently baked in.
2. **Baseline.** Take a read-only snapshot of the target tenant: workspaces, fields, sections, picklists, workflows, roles. Commit it. Every later diff is against this.
3. **Architecture as code.** Describe the target in YAML. Compile it into the formats your apply tool understands. Anything the tool cannot do becomes a handoff task for a human, written so a coding agent could execute it.
4. **Plan and diff.** Produce the planned changes and review them. A single self-contained `plan.html` (versions and diff, progress check-offs, workflow diagrams drawn from the snapshot) works well for customer review.
5. **Apply with a log.** Apply in small steps, log every call, and verify by re-reading.
6. **Hand off.** Give the implementer the manual tasks, scripts and the log.

## Decisions to make with the customer early

- Configure existing workspaces or create new ones. Reconfiguring is usually cheaper and safer.
- Which flows to pilot first, and which user personas you need accounts for. Approval flows cannot be tested with one user.
- What you will never automate. Deleting configuration is a good default for "never".

## API surfaces

- **v3 REST** (`/api/v3/...`) for records, workspaces, fields, roles, groups and permissions.
- **Configuration API** (an undocumented surface used by admin tooling) for creating and migrating workspace configuration. Its base path differs by tenant type, so discover it per tenant: read the tenant configuration keys `ADSK_API_BASE_URL` and `ADSK_FLC_CONTEXT` through `GET /api/v3/configurations/{key}`. Hardcoding the sandbox path breaks on production tenants.
- Plan for tokens that expire after about an hour.

## Things you cannot read or do

Learned the hard way; confirm for your tenant:

- Item lifecycles are not readable by the routes tried (`404` on both the v3 and Configuration paths).
- BOM-level fields were not readable.
- `403 VIEW_BOMS` on a BOM view means the workspace has no BOM tab. It is not an error.
- Workspace-backed picklist values are item data, so a configuration snapshot does not include them.
- Role names must be unique (`409 CONFLICT`).
- Deleting a workspace leaves its roles behind. Clean them up explicitly (`DELETE /api/v3/roles`).
- A required dropdown has a `dropDownSelection` validator, not a `required` one.
- Creating a field by PATCH after a port needs validator value IDs and the source display order, or it fails.

## Test discipline

- Do live checks on a throwaway workspace you are happy to mutate, and restore it afterward.
- Re-read permissions and state after writes. Permission read-back can lag.
- Keep an in-memory fake of the API so the planner and diff logic have fast unit tests.

## Roles and permissions

Export roles as group names only (never user identities) so a role set can move between tenants. Creating roles for a new workspace is a separate, explicitly opted-in step from creating the workspace.
