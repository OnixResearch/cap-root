# Agent Notes

`cap-root-core` is a pure platform-native lexical path crate. It must not open files, inspect metadata, canonicalize ambient paths, call `cap-std`, read environment state, or imply filesystem confinement.

Keep consumer policy explicit. OnixOS and Cairn differ on whether `.` addresses the capability root and on symlink handling. Do not collapse those differences into an ambient default.

Every behavior change requires positive and negative tests. Preserve non-UTF-8 `OsStr` components and platform prefix semantics.

Run:

```sh
nix develop -c cargo fmt --all -- --check
nix develop -c cargo test --workspace
nix develop -c cargo clippy --workspace --all-targets -- -D warnings
nix flake check -L
```

Do not add sibling path dependencies to consumers. Adoption requires a published immutable Git revision and aligned Cargo/Nix source identity.
