<p align="center"><img src="https://raw.githubusercontent.com/go-crdt/brand/main/social/go-crdt.png" alt="go-crdt" width="720"></p>

# go-crdt

**Collaborative editing in pure Go — the same merge logic on the server and in the browser.**

A conflict-free replicated data type settles concurrent edits by the operations
themselves, so nothing has to be authoritative. Three things follow that the
alternative — operational transform, as used by ShareDB and Google Docs — cannot
offer:

- **no server decides who won**, so there is no transform function to get right
  for every pair of operation types;
- **a participant keeps typing with the network down** and reconciles on return;
- **the server can be restarted or replaced** without a handover protocol.

Written in Go so that the browser can run it too. The engine compiles to
`js/wasm`, which means a browser tab and the server execute the *same* merge
implementation rather than two that have to agree — something a JavaScript client
paired with a Go server cannot claim.

## Repositories

| Repo | Role |
| --- | --- |
| [`crdt`](https://github.com/go-crdt/crdt) | The replicated text document: `Doc`, operations, version vectors, snapshots, and ephemeral presence. RGA sequence with tombstones; a per-site sequence number for identity and a Lamport clock for ordering. Zero dependencies. |
| [`collab`](https://github.com/go-crdt/collab) | The gRPC service, server and client that carry a document between people: per-document fan-out, snapshot on join, presence, and a persistence seam. Reaches a browser over `grpc-transports/websocket`. |
| [`brand`](https://github.com/go-crdt/brand) | Logo, favicon and social banner. |
| [`docs`](https://github.com/go-crdt/docs) | Documentation, published at [go-crdt.github.io/docs](https://go-crdt.github.io/docs/). |

## How convergence is proven

Randomised delivery — late, reordered, duplicated — samples the space of
orderings; it does not cover it. So small concurrent histories are also replayed
in **every possible order**, and replicas are compared on their *encoded state*
rather than merely their text, because agreeing on the text is the weaker claim.

Every decoder is fuzzed, which found five states a snapshot could describe that no
replica could ever reach. The end-to-end gate runs **three replicas across two
runtimes** — one native, two compiled to WebAssembly and executed by Node through
a real WebSocket — and requires all three to converge with no character lost.

A skipped test is not a passing one: CI fails that job when the toolchain is
missing rather than quietly turning it green.

## Standards

Pure Go, `CGO_ENABLED=0`. **100% statement coverage** as a CI gate, error branches
included. Validated on all six of Go's 64-bit targets — amd64, arm64, riscv64,
loong64, ppc64le and s390x, the last of which is big-endian and keeps the
deterministic encodings honest. BSD-3-Clause throughout.

📖 **[go-crdt.github.io](https://go-crdt.github.io/)** · **[Documentation](https://go-crdt.github.io/docs/)**
