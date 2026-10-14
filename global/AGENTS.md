> **全局 AGENTS.md 编写要求**: 本文件（~/.dsh/AGENTS.md，zcode/Claude Code 软链接同源）会被加载到每个上下文，必须保持极简。只写**硬约束**，不写最佳实践；只写**触发条件+动作**，不写理由；能用一行就不用一段。

## 文件写入

**Write/Edit 前必须先 Read 目标文件**（同一会话内未读过的文件），避免触发 "File has not been read yet" 错误。

## Search

**分工：结构问题用 codegraph，已知符号/字面量用 rg，最后才 grep**
- 结构/流程问题（执行链怎么串、跨文件数据流）→ `codegraph_explore`
- 已知符号名找消费方/文件清单（如「谁调 pushToRemoteSynced」）→ `rg -l` 直查，**不用** codegraph 自然语言查询（命中宽松、返回噪音大）
- 字面量搜索（文本、注释、日志）→ `rg`
- 带行号 → `rg -n`；**禁止写 `rg -rn`**（`-r` 是替换标志，输出会被替换成 "n"）

## 留作后续 / 遗留工作

- 想留作后续 / PoC 够用 → **必须让用户选**，禁止单方面决定
- 用户确认留作后续，或存在未完成/遗留工作项 → **必须** `/to-spec` 或 `/to-tickets` 沉淀到 issue tracker，禁止仅留在对话/记忆里
- 遗留只写进某工单的 Resolution / 范围外小节 → **不算已记录**：必须开独立 issue（「留待后续统一决策」同样开单），每项遗留都有自己可关闭的 issue

## 开发流程（先讨论方案，再按方案复杂度分流）

- **纯研究/只读探索** → 直接用搜索工具，跳过讨论与 plan/spec/tickets
- **其余任务** → 先 `/grill-with-docs`（或用户点名 `/wayfinder`）讨论方案；讨论收尾 → 给出复杂度判定与路由，用户可改；再按**方案的复杂度**走：
  - **低复杂度**（一句话能改完）→ 直接改
  - **中复杂度**（常规改动/单一决策）→ plan 模式出计划 → 用户批准后实施
  - **高复杂度**（新功能/跨模块/多步骤）→ **标准开发流程**：/to-spec → 审查 spec → /to-tickets → 审查 tickets → grill所有需要讨论的点 → **用户确认** → /implement → /code-review → 问题修复 → 提交 → 提供执行下一个ticket的提示词
- **讨论后仍拿不准复杂度** → 让用户选
- **原型三则**：做原型 → 用户故事地图先行对齐；交互与布局贴生产形态；演示数据真实不伪造

**Tickets→Implement 门禁**：tickets 审查通过后，**必须**让用户确认才可进入 `/implement`，禁止自动推进。

**Implement 并行冲突检测**：进入 `/implement` 时检查并行会话占用，共享同一工作树的并行 → 询问用户（等待 / 换工单）；各线独立 worktree 的并行允许（2026-08-24 拍板，setup 要点与流程记录见 oke 仓 `.scratch/parallel-worktree-log.md`）。

**多工单并行通过子代理执行**：多工单并行编排时，实现/审查/读 diff 全派子代理；主上下文只做编排门禁、短 bash、issue 操作与子代理汇报的压缩转述。

本机是一台处于外网的Mac，无法使用 DevAgent，因此所有的DevAgent相关的真机验证需要在发版（/release-bump）后由开发者在另一台内网中的Windows进行。其余验证分层（2026-08-24 定稿，适用于 oke 仓）：builtin+glm 与 qwen/pi 路径的外网验证在本机做（管线层 agent 自主抽样 + 体验层 HITL，见仓内 docs/tech/external-verification-scheme.md）；内网 Windows = 真实环境最终裁决（旧 git/内网网络/真实仓库/完整用户路径）。

## 任务完成审查

ticket或其他任务改动完成 → build + test 通过 → **必须**在主上下文调 `/code-review` skill 检视（禁止直接派 code-reviewer 等审查类 subagent 代替）→ 审查通过（🟢/🟡）后**直接 commit**，无需等用户确认；🔴 先返工再提交。

`/code-review` 派的两轴并行子代理 = 「独立 subagent 审查」（🔴/🟡/🟢 分级）；派遣 prompt 带 `[SKIP_CHECKS]` 头（照「审查类 Subagent 派遣规则」）；基线取本任务开始前的点，改动未 commit 时用 `git diff <基线>`（含工作树），不用 `...HEAD`。

**工单状态强绑定（GitHub issues）**：实现工单 done = closed——commit 说明写 `closes #N`（可多张），push 时 GitHub 自动关闭；非代码终结（wontfix、needs-info 撤回等）由人工 `gh issue close`，不走 closes。**真机/HITL 验证类工单不持票等验证结果**：实现工单全收后即 `gh issue close`（评论留「拍板关票、发现问题开新工单」），spec/PRD 随末张一并关；验证轮后续发现的问题开新工单走同样路径。push 后**立即**在 issue 评论「已实现 @ <commit hash>，待发版真机验证（内网 Windows）」（夜间 agent 以 assignee 或该评论为防重信号；hash = push 到远端的最终提交，worktree 线 rebase 收线会改写实现 commit 的 hash，push 前的 hash 不可用）。认领 = `gh issue edit <n> --add-assignee @me`，是会话首写动作（并发互斥）。PRD issue 随最后一张工单的 commit（`closes #工单, closes #PRD`）一并关闭，无人工回填动作。`.scratch/` 仅 wayfinder 票（有 map.md 的 feature）适用 claimed/resolved 规则，不走 GitHub。

**commit 前检查暂存区**：`git commit` 提交**整个暂存区**（非本次 add 的文件）；commit 前必须 `git status --short` 确认第一列只有本次要提交的文件，先 `git restore --staged` 清掉他人/无关的预 stage 内容，再 add 自己的 + commit（并发工作树他人预 stage 文件易被误带）。

**主区并行提交用 `--only`**：`git status` 见他人未提交改动 → **不走「restore 清暂存 → commit」路线**（restore 后会被并行会话 re-add 抢回，commit 前竞态窗口封不死），直接 `git commit --only --no-verify <本次文件…> -m`——路径限定原子提交，他人 staged 内容原地保留混不进来；--no-verify 防 lint-staged stash/restore 破坏并行工作树。commit 后必须核对 stat 文件清单恰为本任务的；混入 → `git reset --soft HEAD~1` + `--only` 重做。

**紧急修复豁免**：用户明说紧迫 → typecheck + lint 过即可合入；测试/审查/复验跳过；豁免事实写进 commit message 留痕。

**清理未提交改动先全保**：清理工作树里混入的他人/半成品文件 → 先 `git stash -u` 全量保全再动手；禁止直接 `checkout --` / `rm`（不可逆丢工作）。

**branch 完成即清**：审查通过并 commit 后 → `rebase origin/main` 再 push（线性历史，不产生合并节点）→ 清理 branch 和临时分支，禁止残留。

**跳过重复检查**：主上下文已执行 build/test/lint/typecheck 后，派 subagent 时**必须**传递状态（如环境变量或 prompt 明示），subagent **不得**重复执行已通过的检查。

**完整性检查**：任何任务完成前，必须全局搜索确认所有相关引用处/依赖处已一并修改，不得残留零散尾巴。

## 测试性能

任何 `pnpm test*` 命令单次执行必须 < 60s；超出 → 优化（拆分/并发/缓存/`--onlyChanged`）后再提交，禁止合入慢测试。

测试进程总体内存占用必须 < 3GB；超出 → 优化（限制 worker 并发/拆分/释放资源）后再提交。

## 测试性价比

写每个测试前 **必须** 自问：能防什么 bug？维护成本多高？不划算（防御价值低 / 依赖大量 mock / 易碎难维护）→ 不写；以能防回归为准，不为凑覆盖率写测试。

## 审查类 Subagent 派遣规则

审查类 subagent 派遣 prompt **必须**含四要素，缺一不派：

1. `[SKIP_CHECKS]` 头——prompt 开头明示主上下文已执行的检查（不依赖环境变量，subagent 可能继承不到），格式 `[SKIP_CHECKS] test:passed, lint:passed, typecheck:passed`；subagent 收到标记后**禁止**重复执行相应检查。
2. 审查对象钉死——确切路径 / 命令 / 导出全文贴进 prompt，开放委托（「看看这个仓库」式）不派。
3. 检查单全文内嵌——subagent 没有其他获取渠道。
4. 输出约束——🔴/🟡/🟢 分级 + 每条发现引用原文 + ≤400 词。

## 产物审查（spec / tickets）

spec / tickets **发布到 tracker 之前**，必须在主上下文调专用 skill 审查（🔴/🟡/🟢 分级），🔴 必须返工修完才进下一步。

- **spec 完成后** → 调 `/spec-review` skill（单 architect 子代理；检查单内嵌于 skill，此处不重复维护）。
- **spec 审查通过（🟢/🟡）后无需询问用户** → 直接调 `/to-tickets` 编写工单。
- **工单粒度不 grill 用户**：`/to-tickets` 后不再就粗细 grill 用户，粒度完全交由审查决定。
- **to-spec 测试缝全权委托**：测试缝（复用既有缝/新开纯函数缝）的选择与设计 agent 自定，不问用户；spec 整体审查（/spec-review）照常。
- **tickets 完成后** → 调 `/tickets-review` skill（单 planner 子代理；检查单内嵌于 skill，此处不重复维护）。
- **implement 完成后** → 使用 /code-review skill 进行代码检视。
- skill 派的子代理派遣沿用「审查类 Subagent 派遣规则」四要素（无 build/test 时 [SKIP_CHECKS] 写 `skip:none`）。

## 回答与方案

- **「是否已解决」先查证**：被问某问题是否已修 → 先查现行文件 + git log 再答；对话记忆不算数
- **验收产物贴原文带导览**：请用户验收 LLM 产物 → 贴文件原文并附导览（这是什么文件、该看哪几点）；疑点在原文后单独列出；禁止只给摘要或转述
- **实施方案按 spec 写法**：描述实施方案 → 按 /to-spec 的 spec 结构组织；禁散文式或摘要式概述

## 术语表（CONTEXT.md）

- **词条只写当前定义**：一两句陈述句直说「是什么」；禁历史沿革（改名史/#工单号/「XX 起改为」）与实现细节——这些归 ADR 与 git history；`_Avoid_` 行只列禁用旧名，不写改名故事
- **会话开始先读当前仓 CONTEXT.md**（仓根存在才读，读完再动手）——术语与定义以它为准，每次对话在既有基础上进行

## 对外发布

**禁止向公网 npm 发布任何包**。

**内网发布流程**：import.sh 提取 → git 提交和推送 → 流水线发布。

## Command / Skill / Subagent 开发规范

必须用 `/writing-for-agents` 编写和优化

## 对话风格：对话者是开发者但是不了解实现细节

- **用户视角**：**必须**站在产品的用户视角去解释你的想法，不打比方，不自己创造词语，多用ASCII框图表达。
- **语言**：与用户交流一律使用简体中文（代码/命令/专有名词除外）。
- **句子（ASD-STE100 精神）**：短句，一句只说一件事，主动语态，只用词的本义；长句必须拆。
- **术语一致（ubiquitous language）**：同一事物全篇只用同一个名字，禁止同义词轮换指代（"该机制/此方案/这个设计"混着叫）。
- **新名词先定义再使用**：自造术语/简称首次出现必须紧跟一句话定义；禁止事后补定义（「这是我上轮造的简称」= 违规）。
- **不造新词、只用通用语言**：已有概念禁止新造名词；说话只用行业通用词或平铺直叙表达，拿不准通用词是什么 → 先 WebSearch 查证再用；确需新术语先给用户确认。
- **先定位**：回复第一句先交代结论或当前进展，再展开细节。
- **解释代码与技术方案（ASD-STE100）**：先说改了哪里、为什么改；操作步骤按执行顺序写；关键条件和例外不得省略。

- **解释结构**（按复杂度分级）：
  - 简单（状态查询/一句话答完）→ 直接给结论
  - 中等（常规改动/单一决策）→ 覆盖 Why/What/How 要素即可，顺序与分段自由
  - 大方案（架构/跨模块/多步骤设计）→ 严格 Why→What→How 三段
- **控制信息密度**：每次拷问（grill）用户的时候，只询问一个问题
- **询问方式**：**禁止**使用 AskUserQuestion 工具；直接在对话中提问并给选项让用户选
