---
title: "ProjectKit getting started"
summary: "Install ProjectKit and try a complete usage example."
category: "Guides"
order: 10
keywords: ["project", "workspace", "proposal", "MCP", "unsaved", "revision"]
related: ["index.md", "api.md", "typed-api.md"]
---

# Getting started

Install both `projectkit-0.1.0.buipkg` and `editorkit-0.1.0.buipkg` into `packages/`. Import EditorKit when constructing edit objects.

## Preview and approve an edit

The example registers an unsaved text item, creates a version-checked proposal, and prints its detached JSON preview. In an application, place a review UI between preview and the explicit approval call.

```buoy
Import editorkit
Import projectkit

Sub Main()
    Dim workspace As New ProjectWorkspace()
    workspace.RegisterItem("main", "src/main.bui", "source", "Print \"Hello\"\n")
    Dim item As ProjectItem = workspace.ReadItem("main")
    Dim edits As Array Of EditorEdit = [New EditorEdit(7, 12, "Buoy")]
    Dim changes As Array Of ProjectChangeRequest = [New ProjectChangeRequest(item.Id, item.Version, edits)]
    Dim proposal As ProjectProposal = workspace.Propose(changes)
    Print Json.Stringify(ProposalJSON(proposal))
    workspace.ApproveProposal(proposal.Id)
    Print workspace.ReadItem("main").Text
    workspace.Undo("main")
    Print "Undo: " + workspace.ReadItem("main").Text
End Sub
```

## Keep host state authoritative

Use `ApplyHostEdits` or `ReplaceText` with the exact current version for editor changes. Read the item again after an operation instead of modifying a detached snapshot. `MarkSaved` updates the baseline after the host writes disk; it does not perform the write.

All edit ranges address original UTF-8 byte offsets, with exclusive ends. EditorKit rejects invalid Unicode/CRLF boundaries and overlap. New content, undo, and redo advance revisions; no-op changes preserve revisions and redo. Single-item edits retain per-item history. An approved proposal changing multiple items creates one indivisible history group: Undo or Redo from any changed member moves every member atomically. Later conflicting member history blocks the operation until undone. New edits invalidate connected redo branches; bounded history may conservatively discard older connected groups.

IDs and relative paths are case-sensitive and unique. Paths are labels without empty, dot, dot-dot, backslash, colon, or control components; they are never opened by this package.

[Package overview](index.md) · [API reference](api.md)
