# DSH 多媒体 WebUI 输入支持

简体中文 | [English](README.md)

![DSH Multimedia WebUI Input：文件与文件夹发送、模型读写和安全清理](promo/assets/dsh-multimedia-webui-input-demo.gif)

这是面向当前 DeepSeek Harness Web 客户端的独立社区插件。它在不修改官方
DSH 源码的前提下，为对话输入框增加文件/文件夹选择和拖放。

**Workspace Attachments（工作区附件）** 是当前的实现机制，不是另一个产品名：
WebUI 中选择的资源只会在发送时复制到当前 Agent 工作区，随后 Agent 就能通过
普通工作区工具读取、修改和验证它们。

选择附件时只把浏览器 `File` 对象留在内存；真正点击发送后，DSH 的异步引用
序列化器才通知 Host，把内容流式复制到当前会话工作区：

```text
<cwd>/.dsh/tmp/attachments/<session>/<send>/
```

如果准备失败，消息不会提交，草稿和附件仍可重试。设置页的“多媒体输入与文件管理”可以
按需统计并清理当前会话或当前工作区全部会话；两种删除都要在页面内再次确认，
并且只删除带插件所有权标记的已提交目录。

## 安装与卸载（推荐 NPM）

公开仓库是 [`LCYLYM/dsh-attachments`](https://github.com/LCYLYM/dsh-attachments)。
公开 NPM 包名为 `dsh-multimedia-webui-input`。旧的 scope 名只用于私有预发布分发，
公开安装不依赖它。

先启动官方 DSH Web UI：

```sh
npx --yes @deepseek-ai/dsh web
```

然后在另一个终端把插件加入 `web` profile：

```sh
npx --yes @deepseek-ai/dsh plugin --profile web add dsh-multimedia-webui-input
```

改动 profile 后重启 Web UI。卸载由 DSH 自己完成，且可逆：

```sh
npx --yes @deepseek-ai/dsh plugin --profile web remove dsh-multimedia-webui-input
```

如果 registry 镜像不可用，也可以直接从公开 GitHub 仓库安装：

```sh
npx --yes @deepseek-ai/dsh plugin --profile web add github:LCYLYM/dsh-attachments
```

本仓库仍保留 clone-local 安装器，供源码 QA 或旧的带指纹 DSH checkout 使用；它
不是公开 NPM 安装路径的必需依赖：

```sh
./install.sh
node scripts/install.mjs uninstall
```

安装器会按能力探测目标 checkout，验证组合后的 profile，失败时回滚；不会修改
官方 DSH 源码，也不会删除已经复制到工作区的附件。

## 实现边界

这不是 DOM hook，也不劫持聊天气泡。插件使用 DSH 的正式 Cordis/Web 接口：

- `conversation.input.left`：附件按钮；
- `conversation.input.overlay`：空白新会话的完整附件条；
- `conversation.input.dock`：正式会话附件条；
- 异步 reference serializer：发送瞬间复制，失败阻止提交；
- `settings.section`：统计和清理；
- 同源 Host HTTP 路由：流式上传、提交、统计和删除。

DSH 原生紧凑引用位显示“回形针＋文件名开头”，完整文件/文件夹名、数量和大小
显示在附件条。发送后的用户消息会列出实际相对文件名；完整原始路径到安全路径
映射保存在 `.dsh-workspace-attachments.json`。

## 与现有社区安装生态的关系

官方 DSH profile manager 是安装后端。社区 registry 或 distro 项目可以从根
`package.json` 发现本插件：`dsh.bundle` 表示 profile patch，`dsh.client`
表示 WebUI half；两者都不是运行时依赖。已经退役的 `dsh.plugin.json` 不再作为
第二套公开合同加入。

## 当前状态与宣传素材

当前基线是隔离 `DSH_HOME` 中真实运行的 `@deepseek-ai/dsh@0.1.0-rc.6`。最新版通过
`dsh.client` 发现 WebUI half；同时保留与其内容一致的 legacy `dshClient` 字段，
继续兼容旧扫描器，并由回归测试防止两份声明漂移。根据官方 2026-08-11 命名合同，
触发器依赖使用 `@deepseek-ai/dsh-client-ui-input-trigger`，运行时服务使用
`ctx.inputTriggers`，不再引用已退役的 `ui-slash`/`ctx.slash` 名称。

## 致谢

感谢 [@vlln](https://github.com/vlln) 在私有测试仓库提交两项预发布问题（#1、#2）。
两项反馈促使我们补齐 `dsh.client` 兼容，重新核对 registry 当前的官方 bundle
路径，并完成公开 NPM 包装审查；没有照搬已经退役的独立清单机制。

- GIF 演示已直接展示在本 README 顶部
- [MP4 演示](https://github.com/LCYLYM/dsh-attachments/blob/main/promo/assets/dsh-multimedia-webui-input-demo.mp4)
- [架构与兼容合同](docs/architecture.zh.md)

演示来自隔离的真实 DSH 发送/模型读写/设置清理流程，不是 mock，也没有给产品
加入自动演示模式。
