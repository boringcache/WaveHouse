# WaveHouse cache validation

The `boringcache-validation` branch starts at upstream commit
`1bb38112f0c15953e8fa7eeb721915105ec3da6e`, five first-parent commits before
`f72c8c6f37c88d0a44b4317d2cc905aeb694104f` on September 8, 2026.

Run `BoringCache validation` once for the initial cold and fresh-runner warm
comparison. Then merge each of these upstream commits in order and run
`BoringCache rolling validation` after each push:

1. `4f4204e2860143bd3c3fa1ea053550e9fd5cf66d`
2. `ee49c78cd731d4dae45bd24ac34b40d88e02ad93`
3. `27c22cb4fb9f1aec9e1d93a8554a93f6e36e8ba5`
4. `e203f3a9cf23b05ede4d261772bb23c396ba01b1`
5. `f72c8c6f37c88d0a44b4317d2cc905aeb694104f`

Both providers use Ubuntu 24.04, the Go version declared in `go.mod`, identical
source, and fresh local cache directories. The fork's `main` branch preserves
the upstream tip above; artifacts record the merge base with that snapshot.

The cases measure separate storage:

- `unit` compares GitHub's dependency-keyed Go build archive with BoringCache's
  Go compiler adapter. Both run `make test-unit COV_DEFER=1`, preserving upstream
  `-race`, `-cover`, and the 15-second test timeout. Module downloads remain
  uncached in this case and have a separate timing record. This case does not
  run the TypeScript, integration, or end-to-end suites or the consolidated
  coverage gate.
- `modules` compares module archives, including the complete `go mod download`
  graph. Neither provider caches compiler output in this case. The GitHub key
  retains upstream's `go.mod` and `go.sum` dependency identity and prefix restore
  policy; BoringCache publishes a stable module tag.

The initial namespace is `validation-20260908`. It is deliberately retained
through all five commits. Repeating the cold workflow in that namespace is a
warm repeat and must not be reported as a new cold sample. The second rolling
commit changes a dependency; GitHub's immutable key creates a new archive there.

BoringCache uses GitHub OIDC with `id-token: write`. The workspace connection
must trust this fork, the validation branch, and both workflow files. Cold and
commit jobs publish; warm jobs restore only. No static cache token is required.

Artifacts retain workload timing, source identity, and the Action evidence
available before job cleanup. Final cache publication and proxy diagnostics
occur in the Action post step; retain the complete job logs and BoringCache
run/session evidence when reporting storage, transfer, or reuse. Missing
storage evidence is unavailable, not zero bytes. These runs do not establish
the frequency of eviction in the upstream repository or cross-PR reuse.
