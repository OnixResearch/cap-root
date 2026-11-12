# Consumer adoption status

## Validated boundary

Disposable migrations against exact OnixOS and Cairn base revisions passed 8 and 5 focused tests respectively; Cairn's focused Clippy rail also passed. The pilot preserved each consumer's root-marker, logical-target, internal-link, diagnostic, and capability-I/O policies. See [`PILOT.md`](PILOT.md). These are pilot facts, not consumer release evidence.

## Cutover gate

The configured repository `https://github.com/OnixResearch/cap-root` is not yet a published immutable source. OnixOS and Cairn must not take an ambient sibling path dependency or claim an unpublished local commit as external dependency evidence.

After explicit publication authorization:

1. publish one reviewed immutable commit without changing the AGPL license;
2. pin `cap-root-core` by exact Git revision and align Cargo and Nix source identities in each consumer;
3. migrate OnixOS `cli/src/pure_domains/capability_path.rs` while preserving its stable product error classes, logical-target protocol syntax, leaf API, and rejection of internal symlink targets that resolve exactly to the root;
4. keep OnixOS `cli/src/capability_fs.rs`, `cap-std`, `cap-fs-ext`, no-follow component traversal, file-type policy, atomic publication, and rollback behavior consumer-owned;
5. migrate Cairn's pure `validate_relative_path` behavior from `crates/cairn-cli/src/support/capability.rs` with `AdmissionMode::RelativeOrRoot`, preserving its `PermissionDenied` diagnostic translation and explicit empty-input rejection;
6. keep Cairn root opening, metadata checks, bounded reads, traversal, copy, rename, removal, receipt semantics, and all `cap-std` operations consumer-owned;
7. run standalone tests and focused positive/negative consumer suites in disposable worktrees before authoritative cutover;
8. remove duplicate lexical validators only after behavior and diagnostics match.

Cutover success means both consumers use one exact published revision for lexical admission while retaining independent filesystem authority and claims. It does not transfer OnixOS operator authority, Cairn lifecycle authority, `cap-std` I/O guarantees, or release readiness into this repository.
