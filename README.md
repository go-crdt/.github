# .github

Shared defaults for this organisation.

`default.json` is the Renovate preset every repository here extends with
`{"extends": ["github>go-crdt/.github"]}`.

## A pin that lives in a `run:` block

Renovate's `gomod` manager reads `go.mod` and `github-actions` reads `uses:`. A
version written inside a `run:` step is read by neither, so it is a pin nobody
is ever offered an update for. Two custom managers in `default.json` close that:
one for `go install <module>@vN`, one for `npx [--package] <pkg>@<version>`.

Both skip comment lines, and that is not tidiness. A comment is exactly where a
version gets documented, so a comment holding a valid specifier is the *first*
occurrence in the file — and Renovate rewrites by searching for the text it
extracted. `go-crdt/collab`'s `ci.yml` holds that pair today, the same
`@puppeteer/browsers@3.2.7` twenty-two lines above the command it describes.

`corpus/npx-shapes.yml` is the seven shapes the npx rule was measured against —
five that must match, two that must not. It sits outside `.github/workflows` on
purpose, and its header says how to run it.

Minor and patch Go dependency updates merge themselves once the tests pass.
**The Go toolchain does not.** `go1.27.0` miscompiles on loong64
([golang/go#81000](https://github.com/golang/go/issues/81000), bisected on real
hardware to `3fdac6780b`), and a repository without a loong64 lane sees nothing
wrong with the bump. A library that breaks is one library; a toolchain that
breaks is every architecture at once.
