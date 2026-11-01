---
title: "ProjectKit API"
summary: "Public types and operations supplied by ProjectKit."
category: "Reference"
order: 20
keywords: ["project", "workspace", "proposal", "MCP", "unsaved", "revision"]
symbols: ["ProjectWorkspace", "ProjectItem", "ProjectChangeRequest", "ProjectChange", "ProjectProposal", "ProjectProposalStatus", "ProposalJSON", "ProjectMCPHost", "ProjectWorkspace.Propose", "ProjectWorkspace.ApproveProposal", "ProjectWorkspace.ApplyHostEdits", "ProjectWorkspace.MarkSaved"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# API reference

## Workspace

| Symbol | Contract |
| --- | --- |
| `ProjectWorkspace(historyLimit = 100)` | Create model; history limit is 0 through 1000 per item. |
| `RegisterItem(itemId, relativeName, kind, content)` | Register a saved item at revision 1. |
| `Items()`, `ReadItem(itemId)` | Immutable snapshots including current unsaved Text, Version, IsDirty and ResourceUri. |
| `ApplyHostEdits(itemId, expectedVersion, edits)` | Version-checked EditorEdit transaction; report whether changed. |
| `ReplaceText(itemId, expectedVersion, content)` | Version-checked whole-buffer edit. |
| `Undo(itemId)`, `Redo(itemId)`, `MarkSaved(itemId)` | Per-item editing history with indivisible multi-item proposal undo/redo; saved-baseline updates. |

`ProjectItem` exposes Id, RelativePath, Kind, Text, Version, IsDirty, and ResourceUri. Returned arrays/snapshots cannot mutate the workspace. Limits are 4096 items, 8 MiB UTF-8 text per item, 256 retained proposals, 256 items per proposal, and 4096 edits per item transaction.

## Proposals

`ProjectChangeRequest(itemId, expectedVersion, edits)` describes one item. `ProjectChange` exposes ItemId, ExpectedVersion, OldText, and NewText. `ProjectProposal` exposes Id, Status, and Changes; statuses are Pending, Applied, and Rejected in `ProjectProposalStatus`.

`Propose(requests)` creates a pending preview. `ReadProposal(id)` retrieves it; `ApproveProposal(id)` atomically applies after rechecking versions; `RejectProposal(id)` leaves text alone. `ForgetProposal(id)` releases resolved proposals; pending proposals must first be resolved. Applied/rejected proposals cannot be replayed. `ProposalJSON(proposal)` returns detached JSON for a transcript or review UI.

## MCP adapter

`ProjectMCPHost(workspace).Handle(message)` returns a response JSONItem or Nil for notifications. Create one host per client connection. Initialize and then send notifications/initialized before normal operations. Tools are project_list_items, project_read_item, and project_propose_edits; resources expose current unsaved text at registered item URIs. There is deliberately no MCP approve/apply tool.

Schema/method errors produce JSON-RPC errors; execution failures such as stale revisions produce MCP isError content. Tool notifications cannot create proposals. Bound untrusted wire data and reject raw/escaped NUL before Json.Parse; framing, authentication, and server lifecycle remain host duties.

[Getting started](getting-started.md) · [Package overview](index.md)


[Full typed API: properties, parameters, defaults, and returns](typed-api.md)


## Host-controlled agent auto-allow

`ProjectWorkspace.AutoAllowAgentEdits` defaults to `False`. The host may enable
it from a user control; project tools cannot change it. `ProposeAgentEdits`
prepares a proposal, then calls the same revision-checked `ApproveProposal` used
by manual approval when enabled. Applied multi-item changes form one shared undo
group. Auto-allow requires nonzero history capacity. `TakeAgentProposals` drains
new agent proposal IDs so the host can refresh views or show review cards,
including proposals created by nested scripting calls. This queue is independent
of provider-specific tool events. `project_workspace` reports the active policy.
Changing policy affects new proposals only and never saves files.
