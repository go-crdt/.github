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
| [`collab`](https://github.com/go-crdt/collab) | The service, server and client that carry a document between people: per-document fan-out, snapshot on join, presence, and a persistence seam. A browser gets `collab.WebSocket`, the session's own framing; native peers get `collab.GRPC`. Both are served at once from one server. |
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

Full coverage says every line runs, not that anything would notice if a line
were wrong. So each of the **865 refusals** in the code that reads somebody
else's bytes was deleted in turn, to see whether the suite went red.
**Thirty-four did not**, and are pinned now.

That campaign named *themes* rather than files, and a theme cannot say which
files it covered. So `collab`'s root was swept again **by name**, in five
chunks, and the total was compared with what the package reports rather than
with the sum of the chunks — **245 subjects, 19 survivors, eleven more pinned**.
The eleven are not additive to the thirty-four: this is the same ground, read
properly. Three of them had looked cheaper than they were, including a
`binary.Uvarint` count used as a slice index where an overflowing encoding
makes it **negative**, on the bus a second browser tab writes to.

Some were an error of the wrong kind: a test asserting that *an* error happened
where the code below the deleted guard also failed, differently. Others each had
their own shape — a ten-byte blob manifest claiming a gibibyte it has no chunks
for, a varint count used as an index where an overflowing encoding makes it
negative and the next line panics, an off switch for tombstone collection that no
test could reach.

The widest class is the one no assertion about an *answer* can see. Delete the
bound on a count a peer sent and the input is still refused, by the loop
underneath — but it has already sized an allocation from the claim: **six bytes
asking for 1.3 GB**, and a version vector is the first thing a peer sends. One
more produced no answer at all rather than a failure, because a superseded run
may name any sequence number up to 2⁶² and the suite simply waits.

Two were the same defect in different structures, which is the argument for
sweeping mechanically rather than by inspection: a superseded run carries a
sequence range and nothing else, and two predicates that are asked about every
operation arriving twice were reading its other fields — where a zero looks like
a value. In one, a peer resending a run it no longer holds is told its identity
clashes; in the other, that it is resurrecting a collected key. Both refusals
were already there and neither was held by anything.

Ten are in the **browser half** — the WebRTC peer, the BroadcastChannel bus, the
Web Locks election, the browser's own WebSocket. That half could not be swept at
all until the tool learned to refuse it: a native `go test` does not *compile* a
`js && wasm` file, so deleting a refusal there changes nothing the command can see
and the verdict reads "nothing covers this" when nothing looked. Most of what it
found is what an old browser gets instead of a stack trace in a console nobody is
reading.

The [survivors that were not
defects](https://go-crdt.github.io/docs/latest/methodology/#deleting-every-refusal-not-the-ones-somebody-thought-to-try)
are accounted for one by one, each with the measurement rather than the argument.

📖 **[go-crdt.github.io](https://go-crdt.github.io/)** · **[Documentation](https://go-crdt.github.io/docs/)**
