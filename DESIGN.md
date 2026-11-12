# Extraction decision

## Success contract

The extraction succeeds only if one pure API preserves platform-native paths, represents the real OnixOS/Cairn root-marker difference explicitly, rejects lexical escapes deterministically, supports OnixOS internal-link admission without claiming link-follow safety, and leaves all ambient I/O with consumers.

False completion includes wrapping `cap-std`, accepting only UTF-8, silently treating empty input as a root grant, erasing platform-prefix behavior, forcing one symlink policy on every consumer, or describing lexical normalization as filesystem confinement.

## Approach registry

| Family | Mechanism | Result |
|---|---|---|
| Pure platform path | A `std::path` typed value with explicit admission modes and a separate lexical symlink resolver | Selected and validated by the crate tests |
| Generic no-std automaton | A component-kind state machine plus a separate `std::path` adapter | Rejected: it adds an adapter protocol and ownership surface without improving the filesystem claim; every current consumer already requires `std` and platform-native `OsStr` behavior |
| No extraction | Rely exclusively on `cap-std` path-taking methods | Falsified by duplicated pure admission and normalization policy in OnixOS and Cairn; `cap-std` remains the correct I/O mechanism but does not replace typed consumer-facing lexical modes or internal-link planning |

## Consumer differences preserved

OnixOS rejects `.` for ordinary object paths and separately strips leading logical `/` syntax for installer targets. Cairn accepts `.` as a capability-root marker but rejects empty input. These are represented by separate `AdmissionMode` variants instead of being collapsed into one default.

OnixOS admits selected internal relative symlinks after lexical resolution. Cairn currently rejects symlink escape attempts through its filesystem shell and does not need to adopt internal-link recreation. The shared resolver therefore reports a confined lexical destination—including an exact root destination—but never decides whether a consumer should follow, copy, or recreate the link.

## Adversarial audit

The surviving design was checked against:

- empty input versus explicit `.`;
- root-only logical targets;
- absolute paths and platform prefixes;
- `..` that could normalize inward but is forbidden for ordinary paths;
- internal-link parent steps that stay inside versus escape;
- a link target resolving exactly to the capability root;
- a nonsensical link path at the capability root;
- non-UTF-8 Unix components without transcoding;
- Windows drive and UNC prefixes under Windows-specific tests.

A requested independent VibeThinker audit could not be retrieved (`fetch failed`) despite an HTTP-reachable endpoint. Deterministic tests remain the authority; no model review is treated as evidence.

## Non-claims

This design does not open directories, inspect metadata, follow links, prevent substitution races, guarantee no-follow semantics, authorize roots, or prove that later I/O stays confined. Consumers must pair admitted values with held directory capabilities and their own operation-specific policy.
