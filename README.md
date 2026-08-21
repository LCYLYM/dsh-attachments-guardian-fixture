# DSH Multimedia WebUI Input

[简体中文](README.zh.md) | English

> [!IMPORTANT]
> This is a public test fixture for the DSH Plugin Compatibility Guardian. It may receive automated compatibility changes and must not be published to NPM. The production repository and package remain [`LCYLYM/dsh-attachments`](https://github.com/LCYLYM/dsh-attachments) and `dsh-multimedia-webui-input`.

![DSH Multimedia WebUI Input: file and folder send, model read/edit, and safe cleanup](promo/assets/dsh-multimedia-webui-input-demo.gif)

Independent community plugin for the current DeepSeek Harness Web client. It
adds file/folder selection and composer drag-and-drop without modifying the
official DSH source tree.

**Workspace Attachments** describes the current delivery mechanism, not a
separate product: selected WebUI resources are copied into the active Agent
workspace only at send time, so the Agent can read, edit, and verify them
through ordinary workspace tools.

The plugin deliberately uploads only when the user sends. The Host resolves
the live session workspace, streams files into
`.dsh/tmp/attachments/<session>/<send>/`, and then lets DSH's asynchronous
reference serializer prepend the resulting absolute path to the model message.
If preparation fails, DSH keeps the draft and attachment chip.

The Settings panel includes an on-demand **Multimedia input & file management** section. It
can inspect and clean either the active session or every session in the active
workspace. Both cleanup actions require confirmation and remove only committed
directories carrying this plugin's ownership marker. They never cross into a
different workspace or delete unknown content.

## Install from NPM (recommended)

The public repository is [`LCYLYM/dsh-attachments`](https://github.com/LCYLYM/dsh-attachments).
The public NPM package is `dsh-multimedia-webui-input`. The old scoped name was
used only by the private pre-release distribution and is not required for the
public install.

Run the official DSH Web UI, then add the plugin to its `web` profile:

```sh
npx --yes @deepseek-ai/dsh web
```

In another terminal:

```sh
npx --yes @deepseek-ai/dsh plugin --profile web add dsh-multimedia-webui-input
```

Restart the Web UI after changing the profile. DSH owns the profile directory,
pnpm invocation, dependency reconciliation, and reversible removal:

```sh
npx --yes @deepseek-ai/dsh plugin --profile web remove dsh-multimedia-webui-input
```

The package declares both the official `dsh.bundle` and `dsh.client` faces and
has no production dependencies. It targets the current `@deepseek-ai/dsh`
NPM runtime (`0.1.0-rc.6` at the time of this release), not a private source
checkout or a hard-coded local path.

## GitHub fallback and local QA installer

If a registry mirror is unavailable, DSH can install the same public repository
through pnpm's GitHub spec:

```sh
npx --yes @deepseek-ai/dsh plugin --profile web add github:LCYLYM/dsh-attachments
```

The repository also retains a clone-local installer for source-based QA and
older fingerprinted DSH checkouts. It is not required for the public NPM path:

```sh
./install.sh
node scripts/install.mjs uninstall
```

The installer capability-probes the checkout, verifies the composed profile,
and rolls back on failure. It never patches the official DSH source tree and
does not remove attachment data already copied into workspaces.

## Current status

Implemented and regression-tested against the current official NPM runtime
`@deepseek-ai/dsh@0.1.0-rc.6` in an isolated `DSH_HOME`, with a reversible
local installer, Host upload protocol, composer integration, on-demand cleanup,
and cross-platform Node-only filesystem handling.

This is not an official DSH repository Plugin. It is a dual-face Cordis +
`dsh.client` profile bundle. Current DSH installs it through the native profile
plugin command. The package retains an identical legacy `dshClient` declaration
for older scanners; a regression test keeps both declarations in lockstep. The
current trigger integration uses the renamed
`@deepseek-ai/dsh-client-ui-input-trigger` package and `ctx.inputTriggers`
service from the official 2026-08-11 naming contract.

It does not hook or patch chat DOM. Integration uses official Cordis/Web
surfaces: `conversation.input.left`, `conversation.input.overlay`,
`conversation.input.dock`, asynchronous reference serialization,
`settings.section`, and a same-origin Host HTTP route.

## Installer boundary

The official DSH profile manager is the installation backend. Community registry
or distro projects may discover this package from `package.json` (`dsh.bundle`
for the profile patch and `dsh.client` for the WebUI half), but neither is a
runtime dependency. The retired `dsh.plugin.json` manifest is intentionally not
added as a second public contract.

## Acknowledgements

Thanks to [@vlln](https://github.com/vlln) for reporting
the two pre-release issues (#1 and #2) in the private test repository. The
reports prompted the `dsh.client` discovery fix, the current official-bundle
path, and this public NPM packaging review.

## Local promotional assets

- GIF walkthrough is displayed at the top of this README.
- [MP4 walkthrough](https://github.com/LCYLYM/dsh-attachments/blob/main/promo/assets/dsh-multimedia-webui-input-demo.mp4)

Both were captured from a real isolated DSH send/read/settings flow. The
recording helper is QA-only and does not add a demo mode to the product.
