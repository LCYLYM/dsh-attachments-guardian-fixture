# 架构与兼容合同

简体中文 | [English](architecture.md)

## 产品边界

这是社区 dual-face Cordis 插件，不是官方只允许 Skills/MCP 的 repository Plugin。
选择阶段只保留浏览器 `File`；发送时由 DSH 异步 reference serializer 建立 Host
批次，以两个浏览器上传 worker 流式写入 staging，全部成功后原子发布，再返回给
模型一段以绝对附件根目录开头的紧凑路径说明。

绝对根只出现一次；后续文件名和所有权 manifest 使用相对路径。manifest 保留
完整的原始路径、实际安全路径、大小和类型映射，因此聊天气泡不必重复大量绝对
路径，模型仍可获得完整材料。

Host 始终从 live session 读取 `session.header.cwd`，浏览器不能指定目标目录。最终
目录为：

```text
<session cwd>/.dsh/tmp/attachments/<session key>/<send id>/
```

每个已提交批次都有 `.dsh-workspace-attachments.json` 所有权标记。清理只能删除
带该标记且位于当前工作区附件根下的目录。

## UI 与消息投影

插件不使用 `MutationObserver` 或 DOM 劫持。附件按钮、空白会话附件条、正式会话
附件条和设置页分别进入 DSH 的正式 Slot。DSH 原生固定宽度引用位显示回形针和
文件名开头；完整名称、文件数和总大小显示在插件附件条。

当前 DSH 对已发送用户消息使用纯文本投影，也没有第三方 message renderer Slot；
Assistant Markdown 的安全清洗不允许任意本地绝对路径/file URL。因此第一版不
伪造 Codex 风格聊天链接。发送文本会列出可读的相对文件名，完整映射留在
manifest；DSH 自己的文件工具行继续使用其原生可点击路径。等官方提供消息
renderer 或 local-file action Slot 后，再统一升级用户/AI 气泡。

## 为什么生产依赖为零

Host 只使用 Node 标准库；浏览器端是预构建的 DSH module factory，只消费 DSH
已经提供的 React/runtime/slot 服务。NPM 安装只增加这一个小包，不再引入第二套
运行时、registry daemon 或构建工具。

## 公开 NPM 分发

公开仓库是 `LCYLYM/dsh-attachments`，公开 NPM 包名为
`dsh-multimedia-webui-input`。旧的 scope 名只属于私有预发布分发。官方 DSH
profile manager 负责安装生命周期：

```text
npx --yes @deepseek-ai/dsh plugin --profile web add dsh-multimedia-webui-input
```

DSH 通过 pnpm 将包放入 `$DSH_HOME/profiles/web`，读取包内 `dsh.bundle` patch，
再加入 profile bundle 栈。内容一致的顶层 `dshClient` 声明继续兼容旧扫描器，并由
回归测试防止两份声明漂移。registry 不可用时，文档提供
`github:LCYLYM/dsh-attachments` 作为 GitHub fallback。

clone-local 安装器仍供源码 QA 和旧的带指纹 checkout 使用，但不是公开 NPM 路径的
依赖；它不会修改官方 DSH tracked 文件。它记录的 source digest 和回滚元数据只
用于本地 QA，不是运行时兼容性开关。

## 兼容能力探测

安装要求目标仍提供：

- 通过 `resolvePkgJson` 发现 `dsh.client` 包，同时保留 0806 扫描器所需的
  legacy `dshClient` 声明；
- 当前 `@deepseek-ai/dsh-client-ui-input-trigger` 包和 `ctx.inputTriggers`
  服务（不再使用已退役的 `ui-slash`/`ctx.slash` 名称）；
- 当前 `WebServer`/`ctx.webServer` Host 路由服务（不再使用旧的
  `HttpServerService`/`ctx.httpServer` 名称）；
- `conversation.input.left`、`conversation.input.overlay`、
  `conversation.input.dock`；
- 默认消息提交前的异步 `serializeReference`；
- root scope 的 `settings.section`；
- Host HTTP 最长前缀路由。
- 最新版的原生 profile bundle 组合，以及跟随 Slot 声明生命周期的
  `slots.inject()` 注册。

发布运行包后，安装器执行目标版本对应的真实组合路径，并运行
`dsh --profile web --dump-config`。失败会回滚 profile 注册和运行包。卸载先移除
dependency 与 bundle layer，确认 composed config 已不含插件，才删除安装器自有文件。
兼容性只看能力，不看私有仓库名称或指纹。

## 传输、资源和删除规则

- 沿用 DSH Host/Origin/trusted-host 边界；
- 默认单文件 1 GiB、单次 2 GiB、10,000 文件、64 层；
- request body 带背压流式写入，不整文件进内存；
- 浏览器并发 2、Host admission 4，仅限制附件 I/O，不限制 Agent fan-out；
- 未完成批次永不发布，空闲 staging 用内存表按小时回收，不扫描工作区；
- 只有打开设置页或手动刷新时才异步统计；
- 当前会话清理同时校验 session id 和所有权标记；
- 当前工作区清理跳过 `.staging` 和所有未知/无标记目录；
- 两种删除都要求页面内第二次点击确认；
- 文件系统操作使用 Node 标准 API，面向 Windows、macOS 和 Linux。

## 当前验证基线

在全新的临时 `DSH_HOME` 中通过官方公开 profile manager 对
`@deepseek-ai/dsh@0.1.0-rc.6` 做过真实安装/卸载 smoke test。该版本是本次发布的
验证证据，不是硬编码版本门。
