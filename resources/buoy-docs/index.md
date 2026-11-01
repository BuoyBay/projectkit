---
title: "ProjectKit"
summary: "Expose authoritative unsaved project text through reviewable atomic proposals."
category: "Overview"
order: 0
keywords: ["project", "workspace", "proposal", "MCP", "unsaved", "revision"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# ProjectKit

ProjectKit is an in-memory project model shared by an editor and its agent integrations. It combines stable item identities, unsaved text, monotonic revisions, undo/redo, and explicit edit proposals. Its MCP adapter exposes current project resources and proposal creation without granting agents an apply operation.

It depends on EditorKit. There is no filesystem scan, disk persistence, network listener, process execution, binary item support, or transport authentication. The application explicitly registers eligible text items, supplies transport framing, and owns approval and persistence.

## Approval boundary

`Propose` validates a set of changes and retains a preview without modifying items. The host reviews it, then explicitly calls `ApproveProposal`. Approval rechecks every expected revision and applies all changes atomically. A stale item leaves all items unchanged and the proposal pending. Rebuild or reject such a proposal rather than silently rebasing it.

Workspace objects and MCP hosts are confined to one thread unless externally synchronized. Histories and retained proposals store full text, so choose retention policies for project size.

## Read next

- [Getting started](getting-started.md)
- [API reference](api.md)
