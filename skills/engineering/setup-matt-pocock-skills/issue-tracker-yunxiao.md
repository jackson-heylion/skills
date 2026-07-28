# Issue tracker: Alibaba Cloud DevOps (Yunxiao)

Specs, implementation tickets, bugs, and wayfinding work for this repo live as work items in one Yunxiao project. Use the configured Yunxiao MCP project-management tools for all operations.

Authentication is external to the repo. The MCP server requires `YUNXIAO_ACCESS_TOKEN`; never place that token or any other credential in this file.

## Resolved configuration

Replace every placeholder while running `/setup-matt-pocock-skills`.

| Setting | Value |
| --- | --- |
| Organization | `<organization name>` |
| Organization ID | `<organizationId>` |
| Project | `<project name>` |
| Project/space ID | `<spaceId>` |
| Default assignee | `<user display name>` |
| Default assignee user ID | `<assignedTo>` |
| Spec/map category | `Req` |
| Spec/map work item type | `<type name>` |
| Spec/map work item type ID | `<workitemTypeId>` |
| Ticket category | `Task` |
| Ticket work item type | `<type name>` |
| Ticket work item type ID | `<workitemTypeId>` |
| Bug category | `Bug` or `not configured` |
| Bug work item type ID | `<workitemTypeId>` or `not configured` |
| Required custom fields | `<field ID and configured value pairs>` or `none` |
| Terminal status IDs | `<type ID to terminal status ID mapping>` |
| Category representation | `Bug work item type = bug; Req work item type = enhancement` |
| Readiness representation | `labels` or `markdown marker` |
| Blocking representation | `description fallback` unless a concrete relation-create tool is callable |

Do not leave placeholders in the generated `docs/agents/issue-tracker.md`. Re-run setup when the project, work item types, required fields, or workflow changes.

## Identity and references

Yunxiao exposes two identifiers for a work item:

- **Internal ID** — pass this to `get_work_item`, `update_work_item`, comments, attachments, and `parentId`.
- **Serial number** — the human-facing reference such as `KZGR-351`.

After every create, call `get_work_item` with the returned internal ID and report both identifiers. Preserve both in cross-references. When only a serial number is supplied, call `search_workitems` with the configured `spaceId`, checking each configured category explicitly until the returned `serialNumber` matches; do not assume the serial number is accepted as an internal ID.

## Conventions

- **Create a work item**: call `create_work_item` with `organizationId`, `spaceId`, the configured `workitemTypeId`, `assignedTo`, `subject`, `description`, and `formatType: MARKDOWN`. Include configured required `customFieldValues`. Never omit the assignee.
- **Read a work item**: call `get_work_item` with its internal ID, then `list_work_item_comments` when conversation history matters.
- **Search work items**: call `search_workitems` with `organizationId`, `spaceId`, and `category` explicitly. Use `includeDetails: true` when bodies are needed.
- **Comment**: call `create_work_item_comment`.
- **Edit**: call `update_work_item`. Preserve fields and unrelated labels that the skill is not changing.
- **Apply or remove a triage role**: resolve the canonical role through `docs/agents/triage-labels.md`. In label mode, read the item first and update `labels` with the complete union after adding or removing only the mapped label ID. In marker mode, preserve the body and replace the single `Agent state: <role>` line.
- **Resolve a category role**: use the configured work item category/type (`Bug` maps to `bug`; the configured incoming `Req` type maps to `enhancement`). Do not add a duplicate category label unless this project deliberately configured one.
- **Close**: use `update_work_item` with the terminal status ID recorded for that work item type. Never use a remembered or display-name-derived status ID.

Yunxiao work items are the request surface. Code repositories and merge requests are not part of triage unless this file is deliberately extended with a separate code-management workflow.

## When a skill says "publish to the issue tracker"

For a spec or wayfinder map, create a work item with the configured `Req` type. For an implementation or decision ticket, create a work item with the configured `Task` type.

Apply the configured readiness representation. In label mode, use the immutable label ID from `docs/agents/triage-labels.md`; in marker mode, put `Agent state: ready-for-agent` near the top of the Markdown description. Do not map agent readiness to a Yunxiao business workflow status.

## When a skill says "fetch the relevant ticket"

Resolve the reference to an internal work item ID, call `get_work_item`, and fetch comments. If the reference is a URL, retain the URL in any resulting spec or ticket.

## Parent and blocking relationships

- When tickets come from an existing Yunxiao spec or map, pass that spec/map's internal ID as `parentId` when creating each child.
- Create tickets in dependency order so blockers already have serial numbers.
- Use a native `DEPEND_ON` relation only when a callable tool can create the concrete relation record. `list_work_item_relation_work_item_types` is capability discovery, not relation creation.
- Otherwise, the canonical edge is a `## Blocked by` section containing serial-number references. A ticket is unblocked only when every referenced blocker is in its configured terminal status.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a `Req` work item with **child** `Task` work items.

- **Map**: create a spec/map work item containing Notes / Decisions-so-far / Fog.
- **Child ticket**: create a task with `parentId` set to the map's internal ID. Record its type (`research`, `prototype`, `grilling`, or `task`) in the description or in a configured custom field.
- **Blocking**: follow the parent and blocking rules above.
- **Frontier query**: search open `Task` work items in the configured project, keep children of the map, then drop any item whose `Claimed by:` marker is not `none` or whose referenced blockers are not terminal. Yunxiao requires an assignee at creation time, so assignee presence alone cannot represent a claim.
- **Claim**: create decision tickets with `Claimed by: none` in the description and the configured default assignee. To claim, replace that marker with the current user's ID and set `assignedTo` to that user. This is the session's first write.
- **Resolve**: post the answer as a comment, move the child to its configured terminal status, then append the resulting context pointer to the map's Decisions-so-far.
