# Two-consumer adoption pilot

Date: 2026-07-18

This pilot replaced duplicated lexical path admission in disposable OnixOS and Cairn worktrees with `cap-root-core`. It validates the boundary without placing a sibling path dependency in either authoritative consumer repository.

## Inputs

| Consumer | Base revision | Migrated surface |
|---|---|---|
| OnixOS | `fc6eb30246052a0edf236268eb594552dea62ffb` | `cli/src/pure_domains/capability_path.rs` |
| Cairn | `3d4b09d259fbec68867d66a32a772082efc38961` | `crates/cairn-cli/src/support/capability.rs` |

Both migrations used candidate mechanism revision `ea5a62f29bfde95f8d399ffa30361c6aeed832b3`.

## Baselines

Before the disposable migrations:

- OnixOS capability-path suite: 7 passed, 0 failed.
- Cairn capability-root suite: 4 passed, 0 failed.

The OnixOS Nix invocation required host remote builders and secret signing-key files to be disabled. Its detached worktree also required the repo-documented local `ucan` and `basalt` dependency links.

## Results

### OnixOS

The disposable adapter retained OnixOS's public `CapabilityPath`, error-kind, diagnostic-class, logical-target, leaf, and `InternalSymlinkTarget` surfaces while delegating component admission and relative-link resolution to `cap-root-core`. It explicitly rejects a shared result that resolves exactly to the capability root, preserving OnixOS policy.

```text
8 passed; 0 failed; 0 ignored
```

Command:

```sh
nix --option builders '' --option secret-key-files '' develop path:/home/brittonr/git/OnixResearch/onixos \
  -c cargo test --manifest-path cli/Cargo.toml --lib pure_domains::capability_path::tests::
```

The adapter source diff contained 61 inserted and 107 deleted lines. The extra passing test covers the deliberate consumer rejection of an internal symlink target that resolves exactly to the capability root.

The OnixOS dev shell does not include `cargo-clippy`. A separate Nix Clippy toolchain reached 165 pre-existing repo-wide `-D warnings` failures in unrelated OnixOS files; short-format diagnostics contained no finding for `pure_domains/capability_path.rs`. This prevents a whole-library Clippy pass from serving as pilot evidence. The standalone crate Clippy rail passed, and the migrated OnixOS suite compiled and passed.

### Cairn

The disposable adapter replaced the private component loop with `AdmissionMode::RelativeOrRoot` and retained Cairn's `PermissionDenied` translation, capability-handle shell, `cap-std` operations, metadata checks, bounded reads, traversal, copy, rename, removal, and lifecycle non-claims.

```text
5 passed; 0 failed; 0 ignored
cargo clippy -p cairn-cli --bin cairn -- -D warnings: passed
```

Commands:

```sh
nix develop path:/home/brittonr/git/OnixResearch/cairn \
  -c cargo test -p cairn-cli --bin cairn support::capability::tests::
nix develop path:/home/brittonr/git/OnixResearch/cairn \
  -c cargo clippy -p cairn-cli --bin cairn -- -D warnings
```

The capability shell diff contained 18 inserted and 19 deleted lines, including a new focused test proving that `.` remains distinct from empty, absolute, and escaping input.

## Boundary findings

1. **The explicit modes preserve a real policy difference.** OnixOS ordinary paths use `NonEmptyRelative`; Cairn uses `RelativeOrRoot`; OnixOS installer protocol paths use `LogicalTarget`.
2. **The shared internal-link resolver must not own link policy.** It reports a target resolving exactly to `.` as lexically confined. OnixOS rejects that result in its adapter, while Cairn need not use the resolver.
3. **Platform-native paths are required.** The shared API keeps `PathBuf`/`OsStr` rather than introducing a UTF-8 interchange format.
4. **`cap-std` remains the I/O owner.** Neither pilot moved ambient root opening, no-follow lookup, metadata checks, race handling, file types, publication, rollback, or receipts into this repository.
5. **Consumer adapters are appropriate.** Stable consumer diagnostics and API shapes remain local even when the component automaton is shared.

## Cutover decision

The API boundary is validated by two distinct consumers. Authoritative cutover remains blocked by publication only:

1. publish `cap-root` at an immutable Git revision after explicit authorization;
2. replace disposable path dependencies with exact Git-revision pins and aligned Nix source identities;
3. rerun standalone, focused consumer, and repository-local release rails;
4. commit migrations independently in OnixOS and Cairn.

The disposable worktrees and local path dependencies are test scaffolding, not release evidence or durable product inputs.
