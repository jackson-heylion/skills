# Triage Labels

The skills speak in terms of five canonical triage roles. This file maps those roles to the actual labels used in this repo's issue tracker. Trackers that address labels by immutable ID, such as Yunxiao, must record that ID as well as the display name.

| Label in mattpocock/skills | Label in our tracker | Tracker identifier (optional) | Meaning                                  |
| -------------------------- | -------------------- | ----------------------------- | ---------------------------------------- |
| `needs-triage`             | `needs-triage`       |                               | Maintainer needs to evaluate this issue  |
| `needs-info`               | `needs-info`         |                               | Waiting on reporter for more information |
| `ready-for-agent`          | `ready-for-agent`    |                               | Fully specified, ready for an AFK agent  |
| `ready-for-human`          | `ready-for-human`    |                               | Requires human implementation            |
| `wontfix`                  | `wontfix`            |                               | Will not be actioned                     |

When a skill mentions a role (e.g. "apply the AFK-ready triage label"), use the corresponding label string or tracker identifier from this table as directed by `issue-tracker.md`.

Edit the tracker columns to match the existing labels. Do not invent identifiers.
