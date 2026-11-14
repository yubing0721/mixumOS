# mixumOS 0.1.7 — aarch64 + x86_64

Status: Release Candidate / Ready for Tagging. Target release: 0.1.7. Previous: 0.1.6.

> Chinese translation: [release-notes-0.1.7.zh-CN.md](release-notes-0.1.7.zh-CN.md).

## Highlights (Merged)

- **Window-Scoped Bidirectional Clipboard (Route A).** Route A compositor (`mixcomp`) implements the MAP RFC 0004 clipboard specification: capability handshake, window ownership & ACL verification, 4 MiB frame limits, explicit subscription (`display.clipboard.subscribe`), and compositor echo suppression. In-container Wayland/X11 copy events stream to authorized clients, and clients can inject selections directly into windows.
- **Privilege Broker Hardening & Fail-Closed Security (#81).** Requests via `mixum-sudo` resolve to canonical root-owned absolute paths, verify SHA-256 digests, and run via pinned executable copies. Shells, script wrappers, relative paths, and non-root writable binaries are rejected before approval. The broker requires explicit opt-in (`--enable-privilege`), audits approver identities, enforces rate and queue limits, and refuses to start if `MIXUM_TASK_USER` cannot be mapped (Fail Closed).
- **Snapshot Performance: Manifest Reuse (#82).** Unchanged files match parent snapshot metadata (`dev/ino/size/mtime`) and reuse chunk manifests, bypassing rereads and hashing entirely (300-file no-change snapshots drop from 11.2 ms to 1.3 ms; 50 MiB assets drop from 27.5 ms to 0.2 ms).
- **Agent Installation Fixes (#79, #80).** `mixum-ctl agent install` adapts to npm 12 disabled lifecycle scripts, whitelist dependencies (`keytar`, `node-pty`) for `claude-code` and `gemini-cli`, validates with `--version`, and prioritizes agent binary paths during status checks. Landlock sandbox explicitly allows write access to `$HOME/.opencode`.

## Snapshot Engine

- Packer reuses parent chunk manifests when file metadata (`mtime`, `dev`, `inode`, `size`) is untouched, reducing traversal to directory stats (#82).
- Added snapshot benchmarks and unit tests covering manifest reuse and data integrity.

## Display & Clipboard

- Implemented `mixcomp/src/clipboard.c` to handle Wayland data offers and bridge container surfaces with MAP clients.
- `mixum-agentd` handles `display.clipboard.subscribe`, `display.clipboard.set`, and `display.clipboard.changed`.
- Window ACL checks prevent unauthorized clipboard access across window boundaries.
- Added 4 MiB payload cap, JSON control character escape overflow protections, and echo suppression queues.
- Web console automatically subscribes to clipboard events upon window connection.

## Privilege & Isolation

- Sudo approval broker verifies absolute paths, root ownership, and SHA-256 digests (#81).
- Blocks shell scripts (`sh`, `bash`, `zsh`), wrappers, and group/world-writable files.
- Refuses startup if `MIXUM_TASK_USER` cannot be resolved to valid local UID/GID.
- Sandbox whitelists `$HOME/.opencode` in Landlock rules (#80).

## Verification

- Rust workspace tests pass (`cargo test --workspace`).
- Route A bidirectional clipboard verified via `tests/integration/clipboard-e2e.mjs`.
- Privilege approval, denial, and audit logging verified via `tests/integration/privilege-e2e.mjs`.
- Snapshot manifest reuse performance verified in `rust:latest` container benchmarks.

---

## Pending Changes (Open Pull Requests)

The following changes are implemented in open pull requests and scheduled for the next release:

- **#85: Preserve Program Name in Pinned Executables.** Pinned executables retain their program name inside a dedicated directory (`<pin>/<ULID>/<program>`), resolving rejection by Ubuntu's `require-utility` patch (LP#2137745).
- **#84: Snapshot CAS Batch Fsync via syncfs.** Defer per-chunk `sync_all` and issue a single filesystem `syncfs` at completion (10,000 cold files pack in 0.78s down from 21.7s, a 27.6x speedup).
- **#86: Snapshot Restore Compositor Cleanup & .mart Robustness.** Terminates stale compositor processes before memory restore (`engine.restore`), preventing EGL/DRM resource contention; adds 9 negative tests for corrupted `.mart` headers, checksums, and symlink escapes.

## Contributors

- **yubing0721** ([@yubing0721](https://github.com/yubing0721)) — Route A clipboard, privilege broker hardening, manifest reuse, batch fsync, agent installer fixes
- **AshReo** ([@ashreo](https://github.com/ashreo)) — Code review, architectural alignment, release management
