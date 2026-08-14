# Architecture and compatibility contract

[简体中文](architecture.zh.md) | English

## Product boundary

This is a small community dual-face plugin, not an official DSH repository
Plugin. Selection stays in browser memory. Sending invokes DSH's async
reference serializer, which creates a Host batch, uploads ordinary files with
two workers, commits the complete staging directory, and returns a model-facing
path block whose first line is the absolute attachment root. The root appears
once; file paths and the ownership manifest are relative to it, while the
manifest retains the complete original-to-actual mapping. This keeps the
literal user bubble bounded without weakening the model handoff.

DSH currently renders sent user text literally and exposes no third-party
message-renderer slot. Its Markdown sanitizer also rejects arbitrary local
absolute/file URLs in assistant prose. The plugin therefore does not install a
DOM observer or rewrite chat bubbles: model-facing text stays compact, complete
mapping stays in the manifest, and DSH's existing filesystem tool rows retain
their own clickable path behavior. Codex-style rich local-file citations can be
added safely only when DSH exposes a message renderer or local-file action
slot.

The Host always resolves `session.header.cwd` from the live session. A browser
cannot supply a destination directory. Final data lives under:

```text
<session cwd>/.dsh/tmp/attachments/<session key>/<send id>/
```

Every committed send owns a `.dsh-workspace-attachments.json` marker. Cleanup
may delete only directories carrying that marker.

## Why the package has no production dependencies

The Host half uses only Node standard-library modules. The checked-in browser
artifact is a DSH client-module factory and consumes only DSH-provided runtime
seeds/services. NPM installation therefore adds one small package and does not
install a second runtime, registry daemon, or build tool.

## Public NPM distribution

The canonical public repository is
`LCYLYM/dsh-attachments`; its public NPM package is
`dsh-multimedia-webui-input`. The old scoped name belonged only to the private
pre-release distribution. The official DSH profile manager owns the installation lifecycle:

```text
npx --yes @deepseek-ai/dsh plugin --profile web add dsh-multimedia-webui-input
```

DSH resolves the package with pnpm under `$DSH_HOME/profiles/web`, reads its
`dsh.bundle` patch, and adds the package to the profile bundle stack. The
identical top-level `dshClient` declaration remains for older scanners, with a
regression test preventing the two declarations from drifting. A GitHub spec
(`github:LCYLYM/dsh-attachments`) is the documented registry fallback.

The clone-local installer remains available for source-based QA and older
fingerprinted checkouts. It is not needed by the public NPM path and never
patches tracked files in the official DSH checkout. Its source digest and
rollback metadata are local QA bookkeeping, not runtime compatibility gates.

## Compatibility probes

Installation fails unless the target checkout still exposes all required
capabilities:

- `dsh.client` package discovery through `resolvePkgJson`, with the legacy
  `dshClient` declaration retained for the 0806 scanner;
- the current `@deepseek-ai/dsh-client-ui-input-trigger` package and
  `ctx.inputTriggers` service (the former `ui-slash`/`ctx.slash` names are not
  used);
- the current `WebServer`/`ctx.webServer` host route service (the former
  `HttpServerService`/`ctx.httpServer` names are not used);
- `conversation.input.left`, `conversation.input.overlay`, and `conversation.input.dock`;
- asynchronous `serializeReference` before the default prompt sink;
- the root-scoped `settings.section` contribution slot;
- longest-prefix Host HTTP routing.
- for current DSH, native profile bundle composition and `slots.inject()`
  declaration-lifetime registration.

After package publication, the installer runs the matching real composition
path and `dsh --profile web --dump-config`. Any failure rolls back the profile
registration and package snapshot. Uninstall first removes the dependency and
bundle layer, verifies the composed config no longer contains the plugin, and
only then removes installer-owned files.

Runtime compatibility is capability-based. The private repository URL, its
fingerprint, and the local checkout path never select behavior.

## Transport and resource rules

- Same Host/Origin and trusted-host checks as DSH's browser API boundary.
- 1 GiB per file, 2 GiB per send, 10,000 files, 64 levels by default.
- Raw request bodies stream to file handles with backpressure; no whole-file
  buffering.
- Browser upload concurrency is 2; Host admission is capped at 4.
- An incomplete batch is never published. Idle batches expire from the
  in-memory table after one hour, checked every ten minutes without scanning
  workspaces.
- Usage scans run only while the Attachments settings page is open or the user
  refreshes it; there is no background workspace index.
- Session cleanup verifies the current session id and ownership marker.
  Workspace cleanup stays under the current workspace attachment root, skips
  `.staging`, and ignores every unmarked directory. Both require an explicit
  second in-page confirmation click and use Node filesystem APIs on Windows,
  macOS, and Linux.

## Current baseline

Implemented and smoke-tested against `@deepseek-ai/dsh@0.1.0-rc.6` in a fresh
temporary `DSH_HOME` using the official public profile manager. The tested
version is evidence for the release, not a hard-coded runtime gate.
