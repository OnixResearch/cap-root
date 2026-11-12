# cap-root

Pure lexical admission for paths interpreted beneath caller-owned directory capabilities.

`cap-root-core` preserves platform-native `Path`/`OsStr` components, normalizes current-directory components, rejects ambient roots, platform prefixes, and lexical parent escapes, supports explicit root-marker and logical-target modes, validates leaf names, and resolves relative symlink target text against a link parent without filesystem access.

## Boundary

This repository owns lexical path mechanics only. Consumers retain:

- ambient directory grants and capability-handle lifetime;
- `cap-std`, `cap-fs-ext`, or platform-specific no-follow I/O;
- whether the capability root itself is an admissible operation target;
- whether symlinks are rejected, inspected, copied, or recreated;
- file-type, substitution, race, permission, atomic-publication, and rollback policy;
- product diagnostics, authorization, evidence, and lifecycle claims.

A successful lexical admission proves only that the supplied platform-native component sequence fits the selected lexical mode. It does not prove that a filesystem object exists, that a later path lookup stays confined, that symlinks were not substituted, that a capability was authorized, or that an operation is safe.

## Modes

- `AdmissionMode::NonEmptyRelative` matches consumers that require a non-root relative object path.
- `AdmissionMode::RelativeOrRoot` accepts `.` as an explicit capability-root marker while still rejecting empty input.
- `AdmissionMode::LogicalTarget` strips a leading logical root separator for protocol targets such as `/run/service/state`; it never accepts a platform prefix and does not turn the logical root into ambient authority.

Internal symlink resolution permits `..` only while a lexical stack remains beneath the capability root. A target that resolves exactly to the capability root remains visible as `RootRelativePath::is_root()`, so the consumer must explicitly accept or reject that policy case.

## Verification

```sh
nix develop -c cargo fmt --all -- --check
nix develop -c cargo test --workspace
nix develop -c cargo clippy --workspace --all-targets -- -D warnings
nix flake check -L
```

Positive tests cover normalization, root markers, logical targets, leaf paths, internal symlinks, and non-UTF-8 Unix components. Negative tests cover empty input, ambient roots, parent traversal, nested leaves, absolute or escaping symlink targets, root link paths, and Windows prefixes when built on Windows.

See [`DESIGN.md`](DESIGN.md) for the extraction decision, [`PILOT.md`](PILOT.md) for disposable OnixOS/Cairn migration evidence, and [`ADOPTION.md`](ADOPTION.md) for the authoritative cutover gate.

## License

AGPL-3.0-or-later. See [`LICENSE`](LICENSE).

## References

- [OnixOS](https://github.com/onixcomputer/onix-modules), capability path and internal-link policy at reference revision `fc6eb30246052a0edf236268eb594552dea62ffb`.
- [Cairn](https://github.com/OnixResearch/cairn), capability-relative shell validation at reference revision `3d4b09d259fbec68867d66a32a772082efc38961`.
- [`cap-std`](https://github.com/bytecodealliance/cap-std), the capability-oriented filesystem I/O layer intentionally not duplicated here.
