---
title: "projectkit typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["ProjectChange", "ProjectChangeRequest", "ProjectItem", "ProjectMCPHost", "ProjectProposal", "ProjectProposalStatus", "ProjectWorkspace"]
dependencies: []
symbols: ["ProjectChange", "ProjectChangeRequest", "ProjectItem", "ProjectMCPHost", "ProjectProposal", "ProjectProposalStatus", "ProjectWorkspace"]
related: ["api.md", "getting-started.md", "index.md"]
---

# projectkit typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## ProjectItem

`Public Class ProjectItem`

Immutable snapshot of one registered project item, including its unsaved content.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Id` | `String` | Yes | No | — | — |
| `RelativePath` | `String` | Yes | No | — | — |
| `Kind` | `String` | Yes | No | — | — |
| `Text` | `String` | Yes | No | — | — |
| `Version` | `Integer` | Yes | No | — | — |
| `IsDirty` | `Boolean` | Yes | No | — | — |
| `ResourceUri` | `String` | Yes | No | — | — |

## ProjectChangeRequest

`Public Class ProjectChangeRequest`

Byte ranges address the exact item revision specified here.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `ItemId` | `String` | Yes | No | — | — |
| `ExpectedVersion` | `Integer` | Yes | No | — | — |
| `Edits` | `Array Of EditorEdit` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(itemId As String, expectedVersion As Integer, edits As Array Of EditorEdit)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `itemId` | `String` | `ByVal` | Required |
| `expectedVersion` | `Integer` | `ByVal` | Required |
| `edits` | `Array Of EditorEdit` | `ByVal` | Required |

Returns: no value.

## ProjectChange

`Public Class ProjectChange`

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `ItemId` | `String` | Yes | No | — | — |
| `ExpectedVersion` | `Integer` | Yes | No | — | — |
| `OldText` | `String` | Yes | No | — | — |
| `NewText` | `String` | Yes | No | — | — |

## ProjectProposalStatus

`Public Enum ProjectProposalStatus As Integer`

### Values

| Name | Type | Value / Default | Description |
| --- | --- | --- | --- |
| `Pending` | `ProjectProposalStatus` | `0` | — |
| `Applied` | `ProjectProposalStatus` | `1` | — |
| `Rejected` | `ProjectProposalStatus` | `2` | — |

## ProjectProposal

`Public Class ProjectProposal`

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `Id` | `String` | Yes | No | — | — |
| `Status` | `ProjectProposalStatus` | Yes | No | — | — |
| `Changes` | `Array Of ProjectChange` | Yes | No | — | — |

## ProjectWorkspace

`Public Class ProjectWorkspace`

One-thread in-memory project model. No filesystem or process side effects.

### Constructor

`Public Sub Constructor(historyLimit As Integer = 100)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `historyLimit` | `Integer` | `ByVal` | `100` |

Returns: no value.

### RegisterItem

`Public Sub RegisterItem(itemId As String, relativeName As String, kind As String, content As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `itemId` | `String` | `ByVal` | Required |
| `relativeName` | `String` | `ByVal` | Required |
| `kind` | `String` | `ByVal` | Required |
| `content` | `String` | `ByVal` | Required |

Returns: no value.

### Items

`Public Function Items() As Array Of ProjectItem`

No parameters.

#### Returns

| Type |
| --- |
| `Array Of ProjectItem` |

### ReadItem

`Public Function ReadItem(itemId As String) As ProjectItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `itemId` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `ProjectItem` |

### ApplyHostEdits

`Public Function ApplyHostEdits(itemId As String, expectedVersion As Integer, edits As Array Of EditorEdit) As Boolean`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `itemId` | `String` | `ByVal` | Required |
| `expectedVersion` | `Integer` | `ByVal` | Required |
| `edits` | `Array Of EditorEdit` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Boolean` |

### ReplaceText

`Public Function ReplaceText(itemId As String, expectedVersion As Integer, content As String) As Boolean`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `itemId` | `String` | `ByVal` | Required |
| `expectedVersion` | `Integer` | `ByVal` | Required |
| `content` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Boolean` |

### RenameItem

`Public Sub RenameItem(itemId As String, relativeName As String)`

Renames an item in place, keeping its id, text, saved baseline, version and undo/redo history. Rejects a path another item already uses. The host performs the disk move.

### MarkSaved

`Public Sub MarkSaved(itemId As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `itemId` | `String` | `ByVal` | Required |

Returns: no value.

### Undo

`Public Function Undo(itemId As String) As Boolean`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `itemId` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Boolean` |

### Redo

`Public Function Redo(itemId As String) As Boolean`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `itemId` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Boolean` |

### Propose

`Public Function Propose(requests As Array Of ProjectChangeRequest) As ProjectProposal`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `requests` | `Array Of ProjectChangeRequest` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `ProjectProposal` |

### ReadProposal

`Public Function ReadProposal(proposalId As String) As ProjectProposal`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `proposalId` | `String` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `ProjectProposal` |

### ApproveProposal

`Public Sub ApproveProposal(proposalId As String)`

Host-only approval. Builds all replacement states before a single atomic state swap.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `proposalId` | `String` | `ByVal` | Required |

Returns: no value.

### RejectProposal

`Public Sub RejectProposal(proposalId As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `proposalId` | `String` | `ByVal` | Required |

Returns: no value.

### ForgetProposal

`Public Sub ForgetProposal(proposalId As String)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `proposalId` | `String` | `ByVal` | Required |

Returns: no value.

## ProjectKit functions

### ProposalJSON

`Public Function ProposalJSON(proposal As ProjectProposal) As JSONItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `proposal` | `ProjectProposal` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `JSONItem` |

## ProjectMCPHost

`Public Class ProjectMCPHost`

Stateful MCP request handler. The application supplies byte transport and host approval UI.

### Constructor

`Public Sub Constructor(workspace As ProjectWorkspace)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `workspace` | `ProjectWorkspace` | `ByVal` | Required |

Returns: no value.

### Handle

`Public Function Handle(message As JSONItem) As JSONItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `message` | `JSONItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `JSONItem` |
