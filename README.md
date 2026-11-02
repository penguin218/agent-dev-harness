# agent-dev-harness

一套纯 Markdown 的通用开发流程 harness——**独立控制仓**形态：harness 自身独立成仓，注册 N 个业务仓后纳管（单仓项目 N=1，同一套结构）。AI Agent 读取 [AGENTS.md](AGENTS.md) 即按「需求对齐 → 计划 → 实现 → 验收 → 知识回流」的闭环流程开发，全程留痕、断点可恢复、多人可并行。

## 流程一图流

```text
需求对齐（plan.md + acceptance.md 成文）→ 探索/开发/自测 → 代码自优化自查
→ commit/合并/push/发版/预发部署（需求对齐后自主）→ 按 acceptance.md 自行验证
   ↳ 不通过：归因 → 修复 → 再提交/发布/部署/验证
→ verification.md → 用户最终验收
→ 收口：知识回流 + changelog + backlog + feat/dev-<username> 合入并推送 feat/develop
```

预发是否就绪只看流水线对应部署 stage 是否为 `SUCCESS`，不等待流水线总状态终态。CR/PR 与预发部署必须基于 `feat/dev-<username>` 或重大功能专项分支；生产合入与生产部署由人工完成。

各环节硬性要求见 [AGENTS.md](AGENTS.md)（最高执行规约），生命周期详述见 [docs/process/lifecycle.md](docs/process/lifecycle.md)。

## 接入三步

1. **建控制仓**：把本目录作为独立仓库（或独立目录）放在业务仓之外，git 管理并提交一次初始状态。
2. **注册业务仓**：每个要纳管的业务仓——复制 [templates/project-registry.md](templates/project-registry.md) 到 `config/projects/<repo>.md`（不入库），让 Agent 探索该仓后填写（本机路径/技术栈/clean 预检排除项/自测命令/变更规约表/关键约束）；同时从 [templates/project/](templates/project/) 复制知识库骨架（含 `knowledge/` 空目录）到 `projects/<repo>/` 并初始化 overview.md，在 [projects/README.md](projects/README.md) 看板加一行。单仓项目注册一个仓即可。
3. **开第一个任务**：按 AGENTS.md 的任务命名规则建立 `tasks/<user>-NNNN-YYYYMMDD-<slug>/`，复制 README、plan、acceptance、verification 四件套，从需求对齐开始走完整流程。

## 组织结构

### 目录树

```text
agent-dev-harness/                    # 本体即独立控制仓
├── AGENTS.md                         # 最高执行规约（Agent 接入的唯一入口，先读它）
├── README.md                         # 本文件：接入说明 + 组织结构
├── projects/                         # 项目层（多仓）
│   ├── README.md                     #   全局看板：一行一仓 + 跨仓协议索引（新会话第一入口）
│   ├── backlog.md                    #   跨项目待办池 + 跨会话公告板
│   └── <repo>/                       #   每仓一套知识库（接入时从 templates/project/ 复制）
│       ├── overview.md               #     可覆写快照 + knowledge 索引表（唯一入口）
│       ├── knowledge/<topic>.md      #     主题知识，一主题一文
│       ├── discoveries.md            #     零散发现收件箱（攒到约 30 行拆主题）
│       ├── changelog.md              #     控制仓侧变更日志（与业务仓侧双写）
│       └── architecture.md           #     架构决策（追加型）
├── config/
│   └── projects/<repo>.md            # 每仓注册表（gitignore 不入库：本机路径/技术栈/
│                                     #   预检排除项/自测命令/变更规约表/关键约束）
├── tasks/                            # 任务事实源目录（每任务一个子目录，可跨多仓）
│   └── <user>-NNNN-YYYYMMDD-<slug>/  #   ← 从 templates/task/ 复制四件套填写
├── templates/
│   ├── task/                         # 任务文档四件套模板（README/plan/acceptance/verification）
│   ├── project/                      # 每仓知识库骨架（overview/discoveries/changelog/architecture + knowledge/）
│   └── project-registry.md           # 注册表模板（实例在 config/ 不入库）
├── docs/
│   ├── protocols/                    # 跨仓协议文档（一协议一文；单仓时为空）
│   ├── process/                      # 流程细则：lifecycle
│   └── principles/                   # 原则展开：harness-coding / quality-gates
└── examples/
    └── minimal-task/                 # 填好的最小任务示例（单仓 N=1 形态，仓名 mini-cli）
```

### 四层分工

```text
① 规约层（常量）   AGENTS.md ── 硬性规则：角色/流程/会话铁律/六判据/硬红线（含多人纪律）/事实源
② 参考层（常量）   docs/process + docs/principles + docs/protocols + examples + templates
③ 任务层（流水）   tasks/<user>-NNNN-YYYYMMDD-<slug>/ ── 一任务一目录，README 是事实源，可跨多仓
④ 项目层（沉淀）   projects/（看板 + 每仓知识库 + backlog）+ config/projects/（每仓注册表）
```

- ①② 与 ③④ 分离：规约与细则不随项目变化；③④ 是各业务仓自身的内容。升级 harness 只替换 ①②。
- ③→④ 回流是闭环关键：任务收口时探索结论进 `projects/<repo>/`、变更进 changelog（双写）、遗留进 backlog、认知刷进 overview + 看板行。
- ④→③ 恢复是会话铁律：新会话靠 看板 → overview → knowledge（按索引）→ backlog 四步建立上下文，再开新任务。

### 一个任务里的文件读写顺序

```text
启动    读 projects/README.md 看板 → 目标仓 overview.md → knowledge/（按索引）→ projects/backlog.md
        恢复在途任务时另读其 tasks/<task-id>/README.md 恢复锚点
对齐    写 plan.md（含设计考量六判据 + 涉及仓库/分支）+ acceptance.md（含 Z1-Z4）
开发    读 config/projects/<repo>.md 注册表（预检排除项/变更规约表/自测命令）
        → 逐仓 clean 预检 → worktree 内开发 → 写 README 任务拆分、自测证据
自查    写 README 代码自优化自查表（含触碰文件 wc -l 改前→改后实测）
预发    从个人/专项分支创建 CR/PR 并部署 → 部署 stage SUCCESS 后开始验证
验证    写 verification.md（对照 acceptance 逐项；失败则归因、修复、重新提交/发布/部署/复验）
收口    用户最终验收后写 README 验收记录 → 刷新 overview.md + 看板行 → discoveries.md → changelog（双写）
        → projects/backlog.md → feat/dev-<username> 合入并推送 feat/develop
```

### 各文件的写入方式

| 写入方式 | 文件 | 说明 |
|---------|------|------|
| **不动** | AGENTS.md、templates/、docs/、examples/ | 除 harness 自身升级外不修改 |
| **覆写 / 就地更新** | projects/<repo>/overview.md、config/projects/<repo>.md | overview 整体覆写；注册表在仓事实变化时就地改 |
| **只追加** | projects/<repo>/changelog.md、discoveries.md、architecture.md | 历史不改写（discoveries 拆主题时整段迁移除外） |
| **新增或更新** | projects/<repo>/knowledge/ | 一主题一文；更新后刷文头「最后核准」并同步 overview 索引表 |
| **追加 + 状态更新** | projects/backlog.md、projects/README.md 看板行 | backlog 新条目取最大编号 +1（撞号后提交者顺延；任务编号每人自管不撞）；看板行收尾更新 |
| **随做随写** | tasks/<task-id>/ | 任务期间实时更新，保证中断后仅凭 README 可续作 |

## 分支、授权与多人并行

- **固定分支方向**：日常开发使用 `feat/dev-<username>`；重大功能可使用 `feat/<slug>-<yyyyMMdd>` 专项分支，并先合回个人分支。用户最终验收通过后，按（专项分支 →）`feat/dev-<username>` → `feat/develop` 合并并推送。
- **预发边界**：CR/PR、验证所需发版与预发部署在需求对齐后自主执行，且只基于个人分支或专项分支；不得使用 `feat/develop` 触发预发。
- **生产边界**：禁止直接把成果合入或推送到 `main`/`master`。面向生产的合入与部署由人工审核和执行。
- **多人纪律**：开工前从 `origin/feat/develop` 同步基线并去重任务；任务中只写自己的任务目录；共享文件在收尾集中更新并用 pathspec 限定提交；backlog 兼作跨会话公告板。任务编号按用户独立递增，backlog 编号撞号时由后提交者顺延。

新成员克隆已有控制仓后，`projects/<repo>/` 知识库已随仓获得，但注册表不入库；需按「接入三步」第 2 步为参与的每个仓自建 `config/projects/<repo>.md`，无需重复初始化知识库骨架。

## 从示例开始

不确定一个任务长什么样？先看 [examples/minimal-task/](examples/minimal-task/)——一个填好的自洽示例（单仓 N=1 形态），展示任务 README、plan、acceptance、verification 如何互相引用、逐步写成。
