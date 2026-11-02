# 完整生命周期详述（lifecycle）

> 本文是 [AGENTS.md](../../AGENTS.md) §2 流程的**展开细化**：把一个任务从会话启动到收口拆为七个阶段，每个阶段说明「做什么 / 产出物 / 完成判据」。硬性要求以 AGENTS.md 为准，本文只细化执行动作，不新增、不抬高、不降低要求。

## 七阶段总览

```text
① 会话启动 → ② 需求对齐 → ③ 开发自测 → ④ 代码自优化自查
→ ⑤ 验证与回环 → ⑥ 用户验收 → ⑦ 收口
```

对应 AGENTS.md §2 流程图：①③ 是「探索/开发/自测」的展开，④⑤ 分别对应「代码自优化自查」「按 acceptance 验证」，⑥⑦ 对应「用户最终验收 → 收口」。

---

## ① 会话启动

**做什么**：新会话不假设拥有旧会话记忆（AGENTS.md §3），按铁律四步恢复认知：

1. 读 [projects/README.md](../../projects/README.md) 全局看板
2. 读目标项目 `projects/<repo>/overview.md`：定位、模块地图、关键约定、当前局面
3. 按 overview.md 的 knowledge 索引表**按需**读 `projects/<repo>/knowledge/` 主题文档（不全量读）
4. 扫 [projects/backlog.md](../../projects/backlog.md)：在途事项、遗留债、跨会话公告

若是**恢复在途任务**：先读该任务的 `tasks/<task-id>/README.md`，从头部「恢复锚点」（在途分支 / 当前子任务 / 下一步）接续，不重做已完成的子任务。

**产出物**：无新增文档。恢复在途任务时，若发现恢复锚点过期，先更新锚点再继续。

**完成判据**：能回答三个问题——本项目是什么、当前局面到哪了、本次会话要做什么（新任务 / 续作哪个子任务）。

## ② 需求对齐

**做什么**：动手写代码前的**唯一事前卡点**（AGENTS.md §2 硬要求 1）。与用户对齐一次目标、范围、验收口径；确认后建任务目录 `tasks/<user>-NNNN-YYYYMMDD-<简称>/`（NNNN 取自己名下最大编号 +1，每人自管序列），用 [templates/task/](../../templates/task/) 模板生成文档。

**产出物**：

- `plan.md`（[模板](../../templates/task/plan.md)，必备，**必须含「设计考量」小节**——六判据在规划阶段写清，不涉及也要写「本次不涉及」+ 理由）
- `acceptance.md`（[模板](../../templates/task/acceptance.md)，必备，**必须含 Z 组「代码自优化自查」验收项**，与功能项同权）
- 任务 `README.md`（[模板](../../templates/task/README.md)，开工即建，记录用户确认时间与摘要）

**豁免条件与留痕落点**（AGENTS.md §2 豁免条款）：琐碎改动（typo、单行修复、纯文档、纯配置）可豁免需求对齐与六判据自查，但必须留一句豁免声明——开了任务目录的写入任务 README；未开任务目录的写入对应仓 `projects/<repo>/changelog.md` 条目备注（随改动即时追加，pathspec 限定提交）。豁免要留痕，不是省略。

**完成判据**：用户确认目标/范围/验收口径；plan.md「设计考量」六项无空项；acceptance.md 含 Z 组且逐项可勾验。

## ③ 开发自测

**做什么**：

1. **clean 预检**（硬红线 2，按仓执行）：写代码前执行 `git status --porcelain -- . <排除项>`（排除项见该仓注册表），有任何输出即停止并报告用户，禁止 stash/reset/clean 覆盖用户在途改动
2. **使用固定工作分支**（硬红线 1）：日常工作分支为 `feat/dev-<username>`；重大功能可建 `feat/<简称>-<yyyyMMdd>` 专项分支。禁止在 `feat/develop` 或 main/master 上开发、commit 或 push；业务仓写码在 worktree 内进行
3. 按 plan.md 实施步骤开发；核对注册表 `config/projects/<repo>.md` 变更规约表，触发的条目随本次变更一并完成
4. **README 随做随写**（AGENTS.md §3 四项）：**状态转换即写**（子任务开始/完成/受阻当下更新拆分清单并写子任务总结）、**关键中间产物即写**（探索结论、设计决策、踩坑与修复、验证命令及结果——尤其只存在于会话记忆里的信息）、**恢复锚点常新**（在途 worktree 路径与分支、当前子任务、下一步始终可见）、**子任务完成即小步提交**（用 pathspec 只提交该任务 README，不带入其他改动）
5. **自测并填证据表**：运行注册表登记的自测命令，逐条记入任务 README「自测证据」表（命令 / cwd / exit code / 结果摘要）

**产出物**：个人/专项分支上的实现；任务 README 的拆分清单、子任务总结、探索结论、变更记录、自测证据表。

**完成判据**：plan.md 实施步骤全部完成；自测命令通过且证据表已填；README 可支撑会话中断后仅凭它续作。

## ④ 代码自优化自查

**做什么**：进入验证前必做（AGENTS.md §4）——对 plan.md「设计考量」的**落实复核**：规划时想的六项是否真做到、编码中有没有新冒出触线项。按顺序逐条过六判据（尺寸报警 / 散点数 / 重复实现 / 扩展性 / 分布式 / 高可用），每条给结论 + 动作。尺寸项必须列触碰文件的改动前→改后 `wc -l` 实测表并对照 `projects/<repo>/knowledge/size-debt.md`；⑤⑥ 说明式列出「做了什么」，不要求压测数据；不涉及的写「本次不涉及」+ 理由。详细口径与触线动作见 [docs/principles/quality-gates.md](../principles/quality-gates.md)。

**产出物**：任务 README「代码自优化自查」表（六项结论 + 动作及尺寸行数表）；触线本任务不修的记 [projects/backlog.md](../../projects/backlog.md) 编号；琐碎改动写豁免声明。

**完成判据**：自查表无空项（**空着视为任务未完成**，AGENTS.md §2 硬要求 4）；每个触线项要么已修（有 commit）、要么有 backlog 编号 + 理由。

## ⑤ 验证与回环

**做什么**：代码自优化自查完成后，自主完成实现 commit、个人/专项分支 push、验证所需内部包发版和预发部署，再对照 acceptance.md **逐项实证**——每条验收项按其「验证方式」实际执行，记录证据与结果。CR/PR 与预发部署只能基于 `feat/dev-<username>` 或本任务专项分支，禁止使用 `feat/develop`。预发/日常环境是否生效只看流水线对应部署 stage 是否 `SUCCESS`；部署 stage 成功即可开始验证，不等待 pipeline 或 CR/PR 总状态终态，并记录流水线实例 ID。

**验证不通过**：归因（根因指向代码位置或环境因素）→ 修复 → 重新提交/发布/部署 → 复验，循环直至全部通过。修复 commit 与每轮发布、部署、复验结果均记录在案；失败记录只追加，不得删除或改写。未执行的验证不得写成通过。

**产出物**：`verification.md`（[模板](../../templates/task/verification.md)）——逐项对照表（acceptance 条目 → 验证方式与命令 → 证据 → 结果）+ **失败轮次记录**（现象 / 归因 / 修复 commit / 重新发布部署 / 复验结果，逐轮追加）+ 总结论。

**完成判据**：acceptance.md 全部条目通过（或按其「通过标准」节允许的降级项处理，降级理由与替代证据记入 verification.md）；总结论为「可提交用户最终验收」。

## ⑥ 用户验收

**做什么**：向用户提交 verification.md（附任务 README 等佐证），等待用户最终确认。用户提出异议的，回到对应阶段（需求理解偏差回②，实现问题回③—⑤）处理后再次提交。

**产出物**：无新增文档；用户确认后在任务 README「验收记录」写入确认时间与范围。

**完成判据**：用户明确确认通过。**未验收不得宣称完成**。

## ⑦ 收口

**做什么**：用户最终验收通过后、任务关闭前集中完成六项（AGENTS.md §2.3 + 任务 README 收尾核对清单）：

1. **任务 README 补验收记录**（确认时间与范围），并勾完收尾核对清单
2. **`projects/<repo>/changelog.md` 追加条目**（日期 / commit / 摘要 / 关联任务；业务仓侧已随变更即时追加，双写）
3. **刷新 `projects/<repo>/overview.md`**（快照整体覆写 + knowledge 索引表 + 最后核准时间），并更新 [projects/README.md](../../projects/README.md) 看板行
4. **零散发现归档**：归入 `projects/<repo>/discoveries.md`；同主题攒到约 30 行的拆成 `projects/<repo>/knowledge/<topic>.md`（文头标来源任务与最后核准日期）并登记进 overview 索引表——无索引视为不存在
5. **遗留项登记 [projects/backlog.md](../../projects/backlog.md)**（新条目取最大编号 +1，标优先级与来源任务）
6. **任务分支合并**：重大功能先将专项分支合回 `feat/dev-<username>`，再将 `feat/dev-<username>` 合入并推送 `feat/develop`（merge 不 rebase）。禁止将任务成果合入或推送 main/master；面向 main/master 只能创建 CR/PR 并交由人工审核合入，生产部署也由人工执行。最终验收后可自主删除本任务 worktree 与临时分支

**产出物**：上述六项对应的文件更新与合并 commit。

**完成判据**：收尾核对清单六项全勾；新会话仅凭看板 + overview.md + projects/backlog.md 即可接续项目认知——任务关闭。
