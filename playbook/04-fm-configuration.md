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

The admin UI saves through four channels. Which one a task needs decides whether it can be automated at all. The [PLM docs site](https://autogusx.github.io/fusion-manage-plm-docs/) keeps the endpoint-level detail and is the source of truth; this chapter is the short version. Start with its [Admin UI coverage](https://autogusx.github.io/fusion-manage-plm-docs/guides/admin-ui-coverage/) page, which maps every admin screen to the channel that can do the same job.

| Channel | Where | Good for |
|---|---|---|
| **v3 REST** | `https://<tenant>.autodeskplm360.net/api/v3/...` | Records, users, groups, roles and permissions, section security, reading configuration |
| **v1 Configuration API** | APS gateway, tenant in an `adsk-plm-tenant` header | Creating, updating and deleting workspaces, sections, fields, picklists, categories, tabs, descriptors, relationships, units of measure, delegations and report definitions; cloning workspaces |
| **Legacy JSON** | `/api/rest/...` or `/api/v2/...` on the tenant host, bearer token | Workflow definitions (states, transitions, workflow permissions) and BOM views |
| **Browser only** | `*.do` and `*.form` posts needing the session cookie and an XSRF token | Not usable from an integration: for example BOM fields, sourcing fields, behaviors, print views |

Notes that apply across channels:

- The Configuration API base differs by tenant, so discover it. Read `ADSK_API_BASE_URL` and `ADSK_FLC_CONTEXT` through `GET /api/v3/configurations/{key}`. Hardcoding the sandbox context (`/plm360-ft`) breaks on production (`/plm360`).
- The tenant machine-publishes a route map of every v3 route (Jersey WADL). Use it, plus `OPTIONS` on a route, to check what exists before you probe.
- Plan for tokens that expire after about an hour.
- Everything outside v3 is undocumented. Probe at startup and degrade gracefully if a route 404s.

## Things you cannot read or do

Learned the hard way; confirm for your tenant:

- **Workflows are writable, but whole.** `PUT /api/rest/workflows/{ws}` replaces the complete workflow, so anything you leave out is removed. Read first, change, then send everything. The response from `GET …/{ws}.json` also embeds every script on the tenant with its source, so never log it.
- **BOM views are configurable; BOM fields are not.** You can choose which existing fields a BOM view shows through the legacy `setups` routes. Creating a new BOM field is still browser-only.
- Lifecycle editing has not been mapped.
- Behaviors, print views and item explorer settings are browser-only.
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
