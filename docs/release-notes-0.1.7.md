# mixumOS 0.1.7 — aarch64 + x86_64

Status: pre-release. Target date: 2026-11-14. Previous release: 0.1.6.

> Chinese translation: [release-notes-0.1.7.zh-CN.md](release-notes-0.1.7.zh-CN.md).

## Highlights

- **Window-scoped bidirectional clipboard (Route A).** Route A (`mixcomp`) now implements
  the complete MAP RFC 0004 clipboard contract: capability handshake, window ownership/ACL
  enforcement, 4 MiB frame limits, explicit subscription (`display.clipboard.subscribe`),
  and compositor echo suppression. Text copies inside Wayland/X11 container windows are
  streamed to authorized clients, and client clipboard sets are injected directly into the
  window selection.
- **Privilege broker hardening and fail-closed security.** Privileged commands requested
  via `mixum-sudo` resolve to canonical root-owned binaries with SHA-256 verification and
  pinned copies. Shells, script wrappers, relative paths, and non-root-writable binaries
  are rejected before approval. The broker is strictly opt-in (`--enable-privilege`),
  records approver identities in audit trails, enforces rate/pending limits, and fails
  closed at startup if `MIXUM_TASK_USER` cannot be resolved.
- **Coreutils compatibility in privilege execution.** Pinned executables are preserved
  under program-named directories (`<pin>/<ULID>/<program>`), fixing permission check
  failures on Ubuntu systems carrying the `require-utility` security patch.
- **Snapshot performance: Manifest reuse and syncfs batching.** Unchanged files
  now skip re-reading and chunk hashing entirely by comparing `dev/ino/size/mtime`
  against the parent snapshot manifest (cutting 300-file no-change snapshots from
  11.2 ms to 1.3 ms, and 50 MiB assets from 27.5 ms to 0.2 ms). Cold packing of
  10,000 small files drops from 21.7 s to 0.78 s (27.6× speedup) and restoration
  drops from 22.0 s to 0.50 s (43.6× speedup) through deferred per-object fsyncs
  batched into a filesystem-level `syncfs`.
- **Out-of-box Agent installation fixes.** `mixum-ctl agent install` now supports npm 12
  strict lifecycle script policies for `claude-code` and `gemini-cli` dependencies
  (`keytar`, `node-pty`), validates installations with `--version`, and prepends agent
  binary paths in status probes. Landlock sandbox profiles explicitly permit writes to
  `$HOME/.opencode`.

## Snapshot Engine

- Manifest packer reuses chunk lists from the parent snapshot when file metadata
  (`mtime`, `dev`, `inode`, `size`) matches, converting unmodified file processing into
  fast stat-only traversals.
- Replaced per-chunk `sync_all` during CAS store writes with deferred synchronization,
  issuing a single `syncfs` at the conclusion of pack/unpack operations on Linux.
- Added comprehensive snapshot benchmarks and unit tests covering parent manifest reuse
  and manifest validation.

## Display and Clipboard

- Implemented `mixcomp/src/clipboard.c` supporting Wayland data offers and selection
  routing between container surfaces and MAP display clients.
- Added `display.clipboard.subscribe`, `display.clipboard.set`, and `display.clipboard.changed`
  operations in `mixum-agentd`.
- Window ACL verification ensures clients can only access clipboard streams for
  windows they are permitted to view.
- Added 4 MiB payload protections, JSON string escaping guards, and clipboard echo
  suppression queues.
- Web console UI updated to subscribe to clipboard events upon window connection.

## Privilege and Isolation

- Sudo request broker resolves paths to canonical absolute binaries owned by root (UID 0),
  computes target SHA-256 digests, and stores digests in approval events and audit logs.
- Executable pinning creates isolated, read-only copies under dedicated temporary
  directories retaining the original binary name, avoiding argv[0] spoofing detection.
- Rejects shells (`sh`, `bash`, `zsh`), interpreter wrappers, and world/group-writable
  binaries.
- Hardened daemon startup to refuse execution if `MIXUM_TASK_USER` cannot be mapped to a
  valid local UID/GID.
- Added Landlock sandbox path grants for `$HOME/.opencode`.

## Validation

- Rust workspace tests pass (`cargo test --workspace`).
- Route A bidirectional clipboard verified via `tests/integration/clipboard-e2e.mjs`.
- Privilege broker approval, denial, and audit logging verified via
  `tests/integration/privilege-e2e.mjs`.
- Snapshot manifest reuse and fsync batching verified under `rust:latest` container
  benchmarks.

## Contributors

- **yubing0721** ([@yubing0721](https://github.com/yubing0721)) — clipboard Route A, privilege broker hardening, manifest reuse, fsync batching, agent installation fixes
- **AshReo** ([@ashreo](https://github.com/ashreo)) — review, architecture alignment, release coordination
