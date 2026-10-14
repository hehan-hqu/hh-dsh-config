# DSH 配置同步仓库

本仓库是本机 DeepSeek Harness（DSH）配置的快照。用途：在新机器上恢复配置，或让 agent 读取后自动完成恢复。不含模型与 API 配置（原因见「边界」一节）。

## 仓库内容

| 路径 | 来源（本机绝对路径） | 说明 |
| --- | --- | --- |
| `AGENTS.md` | `~/.dsh/AGENTS.md` | 全局 agent 指令。DSH 会把它注入每个会话 |
| `cordis.patch.yml` | `~/.dsh/cordis.patch.yml` | 全局补丁层。当前只含 codegraph MCP 的自动托管段 |
| `profiles/desktop/` | `~/.dsh/profiles/desktop/` | 桌面端 profile。核心是 `cordis.patch.yml` 与 `package.json` |
| `profiles/web/` | `~/.dsh/profiles/web/` | Web 端 profile。结构与 desktop 相同 |

各 profile 目录内的文件分工：

- `package.json` — 声明插件市场（dshmarket）安装的全部插件，见 `dsh.profile.bundles` 数组。这是「装了哪些插件」的唯一权威清单。
- `cordis.patch.yml` — 用户补丁层。配置 MCP 服务器（glm-search / glm-reader / glm-zread）、内置 web 工具开关、权限预设、UI 偏好。desktop 版已剔除模型相关条目。
- `cordis.yml` — 空入口文件，无需修改。
- `pnpm-workspace.yaml` — pnpm 工作区设置，照抄即可。
- `pnpm-lock.yaml` — 锁定插件精确版本，保证恢复结果一致。

## 恢复步骤（agent 执行）

按顺序执行。命令中的 `$DSH_HOME` 指本机 DSH 主目录，默认为 `~/.dsh`。

1. 复制全局文件：
   ```bash
   cp AGENTS.md $DSH_HOME/AGENTS.md
   cp cordis.patch.yml $DSH_HOME/cordis.patch.yml
   ```
2. 复制两个 profile 的四个文件（`package.json`、`cordis.patch.yml`、`cordis.yml`、`pnpm-workspace.yaml`）到 `$DSH_HOME/profiles/desktop/` 与 `$DSH_HOME/profiles/web/`。若想锁定版本，连同 `pnpm-lock.yaml` 一起复制。
3. 安装插件（在各自 profile 目录内执行）：
   ```bash
   cd $DSH_HOME/profiles/desktop && pnpm install
   cd $DSH_HOME/profiles/web && pnpm install
   ```
4. 设置环境变量 `ZAI_CODING_CN_API_KEY`。三个 glm MCP 服务器的鉴权头都引用它（`cordis.patch.yml` 中的 `!!js` 表达式），没有这个变量 MCP 会 401。
5. 重启 DSH 使补丁层生效。

## 已安装插件清单

来源：`profiles/desktop/package.json` 的 `dsh.profile.bundles`。

| 插件 | 版本 | 用途 |
| --- | --- | --- |
| `@deepseek-ai/dsh-base` | — | 基础包 |
| `@deepseek-ai/dsh-web-app` | — | Web 应用壳 |
| `dshmarket` | ^1.66.14 | 插件市场 |
| `dsh-better-edit` | 1.0.0 | 增强编辑。**已禁用**，`plugins/` 本体未入库 |
| `billion-context` | 0.1.189 | 上下文压缩（ACP） |
| `@wxg-prc-cpg/browser-skill-dsh-plugin` | 0.3.2 | 浏览器技能 |
| `dsh-plugin-message-edit` | 1.2.1 | 消息编辑 |
| `@linxin666/dsh-remote-web-ui` | 0.4.5 | 远程 Web UI |
| `@linxin666/dsh-client-ui-task-board` | 0.4.5 | 任务看板 |
| `@kenz1117/dsh-ui-usage-billing` | 1.4.25 | 用量计费展示 |
| `dsh-mnemon` | 0.5.26 | 记忆系统 |
| `dsh-notify-me` | 1.5.3 | 通知 |
| `@hyzyn/dsh-codegraph` | 0.6.7 | Codegraph 代码图谱 |
| `@michengai/dsh-automation` | 0.1.61 | 定时自动化 |

另有桌面端未列出的 `dsh-better-sidebar`（0.24.1，侧栏增强），在 `dependencies` 中。

## 边界：本仓库不含什么

- 模型与 API 配置。desktop 补丁中的 `agent-default-model`、`llm-pi-ai`、`llm-deepseek-account` 三段已删除。恢复后需在 DSH 设置界面手动配置模型。
- 凭据。`~/.dsh/.credentials.yaml` 与一切 API key 不入库。
- 运行时数据。会话（`sessions/`）、用量账单（`.dsh-usage-*.json`）、日志（`logs/`）、存储（`storages/`）、任务看板数据（`task-board/`）均不含。已禁用的本地插件 `plugins/dsh-better-edit/` 不含。
- `cordis.patch.yml.bak-*` 备份文件不含。

## 更新本仓库

配置变更后，从本机对应路径重新复制并提交。desktop 补丁需再次手工剔除模型相关三段（条目 id：`agent-default-model`、`llm-pi-ai`、`llm-deepseek-account`）。提交前检查 diff 中不得出现 API key 或 token。
