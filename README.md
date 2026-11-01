# Manage project buffers and reviewed edit proposals

An in-memory project model that gives agents the same project items and unsaved
buffers an editor sees. It combines stable item identities, monotonic revisions,
reviewable edit proposals and an MCP request handler. It performs no filesystem,
network or process operations.

Copy both `projectkit-0.1.0.buipkg` and `editorkit-0.1.0.buipkg` into the
consumer's `packages/` directory and use `Import projectkit`. Code constructing
`EditorEdit` values also imports `editorkit`.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add projectkit --git https://github.com/BuoyBay/projectkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.
BPM also resolves `editorkit`.

## Example

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

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer projectkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```

The generated agent files and editor settings stay local and are ignored by Git.

## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
