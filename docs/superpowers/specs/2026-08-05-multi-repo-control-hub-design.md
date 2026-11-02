# agent-dev-harness 多仓多人化改造 设计文档

- 日期：2026-08-05
- 状态：已获用户逐节批准（部署形态 / 多人规则 / 泛化口径 / 目录结构 / 规约改动 / 注册表与实施方式）
- 蓝本：qwenwork-eval-control-hub 控制仓规约（多仓多人原版）

## 1. 背景与目标

agent-dev-harness 是基于 qwenwork-eval-control-hub 规约泛化出的通用开发流程 harness，初版出发点是**单仓库**：harness 整目录复制进目标项目，`project.md` 单项目注册、`knowledge/` 单知识库。泛化过程中丢失了原控制仓的两类能力：

1. **多仓库**：项目注册（config/projects/）、分仓知识库（projects/<repo>/）、全局看板、跨仓协议位
2. **多人并行**：个人分支+集成分支模型、开工前基线同步与任务去重、编号撞号顺延、pathspec 限定提交、backlog 跨会话公告板

本次改造目标：把这两类能力补回通用模板，同时保留模板自身的增量设计（审查闭环 review.md、spec.md、docs/process+principles 细则分层、examples 示例）。

## 2. 已确认的关键决策

| # | 决策点 | 结论 |
|---|--------|------|
| D1 | 部署形态 | **独立控制仓为本体**：harness 自身是一个独立目录/仓库，注册 N 个业务仓纳管；单仓即 N=1 退化情形，同一套结构，不出两套口径 |
| D2 | 多人规则 | **全量收录**：控制仓的多人并行规则泛化后写进规约；单人使用时自然不触发 |
| D3 | 组织特定机制 | **抽象规则 + 注册表参数化**：规约正文只写仓无关的抽象规则，具体分支名、部署/发版命令、CR 工具全部下沉到每仓注册表；推荐约定给默认值（`feat/dev-<user>` → `feat/develop`） |
| D4 | 知识布局 | **按 control-hub 原状抽取**：projects/<repo>/{overview.md, knowledge/, discoveries.md, changelog.md, architecture.md} + projects/README.md 看板 + projects/backlog.md + config/projects/<repo>.md（gitignore 不入库）+ docs/protocols/。曾提出的三个优化（注册表拆两半入库、changelog 单写、看板防膨胀字段规范）**均不采纳**，维持原状 |
| D5 | 结构演进方式 | **在现有模板上演进**（单数改复数），不按控制仓蓝本推倒重建；模板增量设计全部保留 |

## 3. 目标目录结构

```text
agent-dev-harness/                     # 本体即独立控制仓，git 管理
├── AGENTS.md                          # 最高执行规约（改：多仓多人规则并入，见 §4）
├── README.md                          # 接入说明（改：控制仓 + 注册业务仓）
├── projects/                          # ═══ 项目层（control-hub 原状）═══
│   ├── README.md                      # 全局看板：一行一仓（当前焦点/在途分支/最后核准/待办）
│   │                                  #   + 跨仓协议索引 + 每项目文件说明 + 收尾规则
│   ├── backlog.md                     # 跨项目待办池 + 跨会话公告板（从根目录移入）
│   └── <repo>/                        # 每仓一套（单仓时只有一个目录）
│       ├── overview.md                # 可覆写快照 + knowledge 索引表（唯一入口，无索引视为不存在）
│       ├── knowledge/<topic>.md       # 主题知识，一主题一文，文头标来源任务与最后核准日期
│       ├── discoveries.md             # 零散发现收件箱（同主题攒到约 30 行拆成 topic 并移除）
│       ├── changelog.md               # 每仓变更日志（控制仓侧；与业务仓侧双写）
│       └── architecture.md            # 架构决策（追加型）
├── config/
│   └── projects/<repo>.md             # 每仓注册表（gitignore 不入库，见 §5）
├── docs/
│   ├── protocols/                     # 新：跨仓协议文档位（单仓时为空，留 README 说明）
│   ├── process/                       # 保留：lifecycle.md / sdd.md / review-loop.md
│   ├── principles/                    # 保留：harness-coding.md / quality-gates.md
│   └── superpowers/specs/             # 设计规格存档（本文档所在）
├── templates/
│   ├── task/                          # 保留六件套（README/spec/plan/acceptance/review/verification）
│   │                                  #   改：plan.md 加「涉及仓库/分支」；README 变更记录按仓分行
│   ├── project/                       # 新：每仓知识库骨架（overview/discoveries/changelog/architecture）
│   └── project-registry.md            # 新：注册表模板（入库；实例在 config/ 不入库）
├── tasks/NNNN-YYYYMMDD-<简称>/        # 保留（一个任务可跨多仓）
└── examples/minimal-task/             # 保留（引用路径同步更新，体现单仓 N=1 形态）
```

变化归纳：根目录 `project.md`、`knowledge/`、`backlog.md`、`changelog.md` 四个单数项收敛进 `projects/` 与 `config/`，其余原样。

## 4. AGENTS.md 改动清单

其余章节（核心理念、任务体系、审查闭环、六判据、会话铁律主体）不动，新增/修改五组：

### A. 接入模型改写

- 开头「接入方式」：从「复制进目标项目」改为「控制仓独立存在，业务仓通过 `config/projects/<repo>.md` 注册后纳管；单仓项目注册一个仓即可」
- §6 事实源优先级改为：AGENTS.md → `tasks/<task-id>/README.md` → `projects/<repo>/`（项目知识库）+ `config/projects/<repo>.md`（本地实例）→ 业务仓自身规范文件（不得降低本规约红线）

### B. 分支模型抽象化（替换现硬红线 1）

规约正文只写抽象规则，参数下沉注册表：

1. 禁止在主干分支直接 commit/push
2. 每仓注册表声明：`主干分支` / `集成分支` / `工作分支模式`；推荐约定 `feat/dev-<user>` → `feat/develop`，单人小仓可声明集成分支=主干
3. 集成分支只做基线汇聚：禁止在其上开发或 commit；CR/部署类验证动作只基于工作分支，禁止用集成分支触发（多人改动汇聚，失败无法归因）
4. 验收收口时「工作分支 → 集成分支」合并 + 推送是需求对齐即授权的自主动作（merge 不 rebase）；成果不得长期只停在工作分支（否则他人拉不到、基线断裂）。面向主干的合入始终人工
5. 业务仓写码一律 worktree 隔离（多仓多人下升为硬要求），主仓目录保持 clean；worktree 验证期间可保留复用，最终验收后删除

### C. 多人协作纪律（新增一节）

1. **开工前基线同步 + 任务去重**：控制仓 merge 远程主干进个人分支；业务仓 merge 集成分支进工作分支；冲突先解决再动工。同步后扫 `tasks/`、`projects/backlog.md`、看板在途分支确认任务不重复；发现重叠先停下协调（拆分/合并/接手/改号）再开工
2. **编号撞号顺延**：任务编号与 backlog 编号均取现有最大 +1；并行会话撞号，后提交者顺延改号
3. **共享文件收尾集中更新 + pathspec 限定提交**：任务进行中只写自己的 `tasks/<task-id>/`；看板/overview/backlog/changelog 只在收尾集中更新一次，更新前 `git status` 检查他会话未提交改动——只提交自己的文件；暂存区有他人内容时必须 `git commit -- <自己的路径>`，禁止裸 `git commit`
4. **backlog 即跨会话公告板**：占号声明、冲突协调、新增问题条目允许随时追加（不受收尾集中更新限制），pathspec 限定提交；另一会话收尾时必读
5. **基线前进合并**：任一任务分支合入某仓集成分支后，该仓其他在途任务分支应在下次继续开发前（最迟进入下一轮验证前）把集成分支 merge 进自己的任务分支；冲突当场解决，不用 rebase

### D. clean 预检按仓执行（改硬红线 2）

预检对象是「本次任务要写的业务仓」；命令为 `git status --porcelain -- . <排除项>`，排除项由该仓注册表声明。有输出即停止并报告用户，禁止 stash/reset/clean 覆盖在途改动。

### E. 多仓任务口径（§2.1 补充)

- 一个任务可涉及多仓；`plan.md` 必须列「涉及仓库/分支」
- 任务 README 变更记录表按仓分行（仓库列为首列）
- 跨仓同源逻辑落六判据③口径：无法共享的至少两侧注释互指 + 自查节声明「同源双份，改动需双改」

## 5. 每仓注册表设计

实例：`config/projects/<repo>.md`（gitignore 不入库，本机路径各人不同）；模板：`templates/project-registry.md`（入库）。字段：

| 分组 | 字段 | 说明 |
|------|------|------|
| 基本信息 | 仓名 / 本机路径 / 类型 / 技术栈 | 原 project.md 同款 + 本机路径 |
| 分支模型参数（新） | 主干分支 / 集成分支 / 工作分支模式 / worktree 根目录 | 规约 §4-B 抽象规则的参数来源 |
| clean 预检排除项（新） | 如 `':!tmp' ':!logs'` | 硬红线 2 按此执行 |
| 自测命令 | 原 project.md 同款 | 完成任务必须执行并记证据 |
| 变更规约表 | 原 project.md 同款 | 触发条件 → 必须伴随的变更 → 验证方式 |
| 验证性动作清单（新） | 需求对齐即授权的动作及命令（预发部署 / 发版等） | 授权边界以此为准；没有就留空 |
| 关键约束 | 原 project.md 同款 | 从业务仓自身规范提取的硬性限制 |

## 6. 实施方式

1. 工作仓：`/Users/xuanfeng.lhx/Downloads/agent-dev-harness`（已是 git 仓），任务分支 `task/0001-multi-repo-control-hub`，不碰 master
2. 改动文件清单：
   - 改：`AGENTS.md`（§4 五组）、`README.md`（接入三步、目录树、四层分工、读写顺序全部按新结构重写）
   - 新建：`projects/README.md`（看板模板）、`projects/backlog.md`（原根 backlog.md 迁移）、`templates/project/`、`templates/project-registry.md`、`config/`（含 .gitignore 规则）、`docs/protocols/README.md`
   - 删（内容迁移后）：根 `project.md`、`knowledge/`、`changelog.md`、`backlog.md`
   - 一致性过一遍：`docs/process/*`、`docs/principles/*`、`templates/task/*`、`examples/minimal-task/*` 中所有 `knowledge/`、`project.md` 路径引用改为新路径
3. `examples/minimal-task/` 体现单仓 N=1 形态，证明单仓场景不增负担
4. 完成后按模板自身流程走审查（review.md）与验证（verification.md）

## 7. 非目标

- 不改六判据、审查闭环、SDD、生命周期等流程内核
- 不引入 control-hub 的组织特定内容（a1/Aone、pip artlab、具体人名分支）
- 不出 single-repo / multi-repo 两个变体模板
- 不采纳三个候选优化（注册表拆两半入库、changelog 单写、看板字段规范），维持 control-hub 原状
