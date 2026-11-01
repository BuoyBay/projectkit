# ProjectKit · 0.1.0

An in-memory project model that gives agents the same project items and unsaved
buffers an editor sees. It combines stable item identities, monotonic revisions,
reviewable edit proposals and an MCP request handler. It performs no filesystem,
network or process operations.

Copy both `projectkit-0.1.0.buipkg` and `editorkit-0.1.0.buipkg` into the
consumer's `packages/` directory and use `Import projectkit`. Code constructing
`EditorEdit` values also imports `editorkit`.

## Host application API

| API | Behavior |
| --- | --- |
| `ProjectWorkspace(historyLimit = 100)` | Create a one-thread project model; per-item history limit is 0..1000. |
| `RegisterItem(id, relativeName, kind, text)` | Explicitly register a text-bearing project item. Its first revision is 1 and it initially counts as saved. |
| `Items()`, `ReadItem(id)` | Return immutable `ProjectItem` snapshots with `Id`, `RelativePath`, `Kind`, `Text`, `Version`, `IsDirty` and `ResourceUri`. Text includes unsaved host edits. |
| `ApplyHostEdits(id, expectedVersion, edits)` | Apply `EditorEdit` ranges against an exact revision. Returns whether content changed. |
| `ReplaceText(id, expectedVersion, text)` | Whole-buffer host edit with the same revision check. |
| `CanUndo(id)`, `CanRedo(id)` | Query history availability without changing text, versions or history; a later change to another transaction member blocks availability. |
| `Undo(id)`, `Redo(id)` | Undo/redo the next history transaction affecting this item. Multi-file approved proposals move all changed members atomically; return `False` when unavailable. |
| `MarkSaved(id)` | Update the saved-content baseline only; the host separately performs any disk write. |
| `ReloadItem(id, content)` | Take new text and saved baseline after the host rewrote the file itself; records no history and forgets the item's (and its transactions') history. |
| `RenameItem(id, relativeName)` | Rename/move an item in place, keeping its id, text, saved baseline and undo/redo history; the host performs the disk move. |
| `Propose(requests)` | Validate all changes and return a pending `ProjectProposal` without changing any item. |
| `ReadProposal(id)` | Retrieve a retained proposal for review. |
| `ApproveProposal(id)` | Explicit host approval; recheck every revision and apply all changes atomically. |
| `RejectProposal(id)` | Mark a pending proposal rejected without changing any item. |
| `ForgetProposal(id)` | Release an applied/rejected proposal; pending proposals must first be resolved. |

A request is `ProjectChangeRequest(itemId, expectedVersion, edits)`, where
`edits` is an array of immutable `EditorEdit(startByte, endByte, replacement)`
objects. All ranges address the original item text in zero-based UTF-8 bytes;
ends are exclusive. EditorKit validates codepoint/CRLF boundaries, overlap and
batch ordering. The host can use a temporary `EditorDocument(item.Text)` for
UTF-16 position conversion when adapting an LSP or agent protocol.

`ProjectProposal.Id` is stable within its workspace. `Status` is one of
`ProjectProposalStatus.Pending`, `Applied` or `Rejected`. `Changes` returns
immutable `ProjectChange` previews with `ItemId`, `ExpectedVersion`, `OldText`
and `NewText`. `ProposalJSON(proposal)` creates a detached JSON representation
for a transcript, diff view or host UI. Arrays returned by the model are copies.

Approval first builds all replacement states and histories, then swaps the
workspace state once. A stale revision in any item leaves every item unchanged
and the proposal pending, so the host can display the conflict and reject or
rebuild the proposal. Applied/rejected proposals cannot be replayed. Each changed
item receives one history frame sharing the transaction's identity. `Undo(id)`
and `Redo(id)` operate on the whole approved transaction even when invoked from
another member. `ForgetProposal` releases the preview without removing its undo
history. Later edits in a member must be undone before the older transaction can
move; a blocked operation throws `InvalidArgumentException` and changes no text
or revision. The proof uses history identity, so independently recreating the
same bytes does not bypass this gate.

A new edit after undo invalidates redo branches connected through those shared
transactions; unrelated items keep their redo history. If the per-item history
limit evicts a transaction, its frame and older dependent prefixes are pruned
from every participant, preserving newer complete transactions and avoiding
partially undoable groups. Group metadata retains item IDs, not additional full
text copies; ordinary full text frames remain bounded by the configured limit.

Versions advance for every affected member on content changes, undo and redo,
and do not reset when text returns to an earlier value. Saved baselines remain
independent. No-op proposal members preserve revisions and unrelated redo.

See [the runnable example](../examples/projectkit_demo.bui) and
integration tests.

## MCP interface

`ProjectMCPHost(workspace).Handle(message)` consumes an already-parsed JSON-RPC
message and returns a response `JSONItem`, or `Nil` for notifications. Create one
host per client connection. A transport such as AgentKit's JSONL codec or an
application-owned socket supplies framing; this package does not open a listener
or launch an agent. Before parsing untrusted wire data, the transport must bound
message size/depth and reject literal or escaped NUL characters. Buoy's JSON
parser can discard an escaped `\u0000`, so that check belongs before `Json.Parse`;
AgentKit's guarded JSONL decoder supplies this boundary.

The implementation follows MCP lifecycle, tools and resources from the
[2025-11-25 specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/lifecycle).
It negotiates 2025-11-25, 2025-06-18 or 2025-03-26. An unknown requested version
receives 2025-11-25; the client must verify that it supports the returned version.
The client sends `initialize`, then `notifications/initialized`, before normal
operations. `ping` is supported. Capabilities advertise tools and resources
without subscriptions or change notifications.

| MCP operation | Result |
| --- | --- |
| `tools/list` | Exact schemas and annotations for the three project tools below. |
| `project_list_items` via `tools/call` | IDs, paths, kinds, current versions, dirty state and resource URIs. |
| `project_read_item` via `tools/call` | Same metadata plus authoritative current text; requires `itemId`. |
| `project_propose_edits` via `tools/call` | Pending proposal preview; requires `changes` containing `itemId`, `expectedVersion` and `edits` with `startByte`, `endByte`, `text`. |
| `resources/list` | Registered item resources at `buoy-project://item/<id>`. |
| `resources/read` | Current unsaved text for an exact registered URI. |

There is deliberately no MCP apply/approve operation. The application host owns
proposal approval. Tool calls without request IDs are ignored, so notifications
cannot create proposals. Method/tool names and schema keys are case-sensitive. Unknown methods, invalid schemas, unsafe numeric values
and unsupported pagination cursors produce JSON-RPC errors. Tool execution errors
such as stale revisions or unknown items produce MCP `isError` content. Successful
tool content is JSON encoded in a text block, usable by all negotiated versions.

## Scope and bounds

Items represent source files, declarations or other text-bearing project entries;
`kind` is application-defined metadata. The host explicitly registers eligible
items. There is no filesystem scanning, symlink resolution, disk persistence,
binary item support, arbitrary command execution, token management or remote
transport authentication in this package.

IDs use ASCII letters, digits, hyphen and underscore. Project paths are canonical
relative slash-separated names, with no empty, `.` or `..` components, backslash,
colon or control characters. IDs and paths are case-sensitive and unique within
the model. They are labels, not paths that this package accesses on disk.

Limits are 4096 registered items, 8 MiB UTF-8 text per item, 256 retained proposals,
256 items per proposal and 4096 edits per item transaction. Histories retain full
text snapshots and proposals retain old/new text, so applications should choose a
history limit and proposal retention policy suitable for their project sizes.
Confine each workspace and MCP host to one thread or synchronize externally.

## Verification


Maintainers run the private validation suite before release. See the repository README for public build instructions.

