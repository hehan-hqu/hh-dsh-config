# AGENTS.md

本仓库是本机 DSH（DeepSeek Harness）配置的同步快照。对它做的操作几乎只有一种：把本机 `~/.dsh/` 下的最新配置复制进来并推送。详细结构见 README.md。

## 结构

- `global/` — 全局层。`AGENTS.md` 对应 `~/.dsh/AGENTS.md`；`cordis.patch.yml` 对应 `~/.dsh/cordis.patch.yml`
- `profiles/desktop/`、`profiles/web/` — 两个 profile，各含 `package.json`（插件清单）、`cordis.patch.yml`（用户补丁）、`cordis.yml`、`pnpm-workspace.yaml`、`pnpm-lock.yaml`
- 根目录只放 README.md 与本文件（仓库自身的 agent 指令），不放同步下来的配置文件

## 更新规则

- 同步 desktop 的 `cordis.patch.yml` 时，必须删除模型相关条目（id：`agent-default-model`、`llm-pi-ai`、`llm-deepseek-account`），并在文件头保留说明注释
- 以下内容禁止入库：API key、token、`~/.dsh/.credentials.yaml`、会话（`sessions/`）、用量账单（`.dsh-usage-*.json`）、日志、`storages/`、`task-board/` 数据、`*.bak*` 备份、已禁用的本地插件（`plugins/dsh-better-edit/`）
- 基础包（`@deepseek-ai/dsh-base`、`@deepseek-ai/dsh-web-app`）无配置文件，README 插件清单不列
- 提交前必须检查 diff：不得出现密钥类字符串；desktop 补丁不得含模型条目
- 模型配置变更（换默认模型、换 provider）不属于本仓库的同步范围，忽略即可

## 沟通

- 与用户交流用简体中文；短句，先结论后细节
