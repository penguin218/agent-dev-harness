# agent-dev-harness 多仓多人化改造 实现计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 把 agent-dev-harness 从「复制进单个项目」的单仓模板，改造为「独立控制仓 + 注册 N 个业务仓」的多仓多人 harness（单仓即 N=1 退化情形）。

**架构：** 全部是 Markdown 文档改造。根目录四个单数项（project.md / knowledge/ / backlog.md / changelog.md）收敛为 `projects/`（看板 + 每仓知识库）与 `config/projects/`（每仓注册表，gitignore 不入库）；AGENTS.md 并入分支模型抽象规则与多人协作纪律（新增硬红线 6-8）；其余 docs/templates/examples 做路径一致性修订；README 按新形态重写。

**技术栈：** 纯 Markdown + git。无构建、无测试框架——每个任务的「验证」是 grep 断言（陈旧引用清零、新文件存在、链接可解析）。

**规格：** [docs/superpowers/specs/2026-08-05-multi-repo-control-hub-design.md](../specs/2026-08-05-multi-repo-control-hub-design.md)（已获用户批准）

**工作环境：** 仓库 `/Users/xuanfeng.lhx/Downloads/agent-dev-harness`，分支 `task/0001-multi-repo-control-hub`（已存在，设计文档已在其上 commit 42f7bf0）。所有命令的 cwd 均为仓库根目录。

**执行约定：**
- 术语「注册表」= `config/projects/<repo>.md`（每仓一份，不入库）；「看板」= `projects/README.md`。
- 修改现有文件时，先 Read 再 Edit；下文给出的 old_string 摘自当前文件（commit 42f7bf0 状态），若执行时不匹配，以「语义等价处」为准手工定位。
- 每个任务结束都 commit（信息见各任务最后一步），不攒批。

---

## 文件结构总览

| 动作 | 文件 | 职责 |
|------|------|------|
| 新建 | `projects/README.md` | 全局看板：一行一仓 + 跨仓协议索引 + 每项目文件说明 + 收尾/启动规则 |
| 迁移+改 | `backlog.md` → `projects/backlog.md` | 跨项目待办池 + 跨会话公告板（补「项目」列与撞号顺延规则） |
| 新建 | `templates/project/overview.md` | 每仓快照模板（由根 `knowledge/overview.md` 演进） |
| 新建 | `templates/project/discoveries.md` | 每仓发现收件箱模板（由根 `knowledge/discoveries.md` 演进） |
| 新建 | `templates/project/changelog.md` | 每仓变更日志模板（由根 `changelog.md` 演进，补双写口径） |
| 新建 | `templates/project/architecture.md` | 每仓架构决策模板（新增） |
| 删除 | `knowledge/`、`changelog.md` | 内容已迁入 `templates/project/` |
| 新建 | `templates/project-registry.md` | 注册表模板（由根 `project.md` 演进 + 分支模型参数等新字段） |
| 新建 | `config/projects/.gitkeep`、`.gitignore` | 注册表实例目录（实例不入库） |
| 删除 | `project.md` | 内容已迁入 `templates/project-registry.md` |
| 新建 | `docs/protocols/README.md` | 跨仓协议文档位说明 |
| 修改 | `AGENTS.md` | 五组改动：A 接入模型 / B 分支抽象 / C 多人纪律 / D 预检按仓 / E 多仓任务口径与路径 |
| 修改 | `templates/task/plan.md`、`templates/task/README.md` | 涉及仓库/分支表、变更记录仓库列、收尾清单新路径 |
| 修改 | `docs/process/lifecycle.md`、`sdd.md`、`review-loop.md`、`docs/principles/quality-gates.md`、`harness-coding.md` | 路径一致性修订 |
| 修改 | `examples/minimal-task/README.md`、`verification.md` | 引用新路径，标注单仓 N=1 形态（示例仓名 `mini-cli`） |
| 重写 | `README.md` | 接入说明/目录树/四层分工/读写顺序全部按新形态重写 |

---

### 任务 1：projects/ 骨架——看板 + backlog 迁移

**文件：**
- 创建：`projects/README.md`
- 迁移+修改：`backlog.md` → `projects/backlog.md`

- [ ] **步骤 1：git mv 迁移 backlog**

```bash
mkdir -p projects && git mv backlog.md projects/backlog.md
```

- [ ] **步骤 2：改写 projects/backlog.md**（补「项目」列 + 撞号规则；整体覆写为下述内容）

```markdown
# backlog（跨项目待办池 + 跨会话公告板）

**规则**：

- 任务收尾时把遗留项收进此处；启动新任务前扫一眼。
- 新条目取全表最大编号 +1；**编号撞号（并行会话同时取号）后提交者顺延改号**。
- 优先级：P0 阻断 / P1 尽快 / P2 择机。
- 状态：⬜ 未处理 / 🔄 进行中 / ✅ 完成。
- 本文件是**跨会话公告板**：新增条目、状态更新、占号声明、冲突协调可随时追加，无需等到任务收尾（提交用 pathspec 限定，见 AGENTS.md 硬红线 8）。
- 完成的条目移到文末「已完成」节，并注明来源任务（`task#NNNN`）。

## 待办

| # | 项目 | 事项 | 优先级 | 来源 | 状态 |
|---|------|------|--------|------|------|

## 已完成

| # | 项目 | 事项 | 来源 | 完成任务 |
|---|------|------|------|----------|
```

- [ ] **步骤 3：新建 projects/README.md**（全文如下）

```markdown
# 项目知识库 — 全局看板

> 新会话第一入口：先读本页看板，再进目标项目的 `<repo>/overview.md`（会话铁律，AGENTS.md §3）。

## 看板

> 一行一仓；任务收尾时更新本仓所在行。单仓项目（N=1）本表只有一行。

| 项目 | 当前焦点 | 在途分支 | 最后核准 | 待办（backlog） |
|------|----------|----------|----------|-----------------|
| `<repo>`（接入时替换本示例行） | `<在做/刚验收的任务一句话（可引 task#NNNN）>` | `<工作分支（commit 短 sha）>` | `<YYYY-MM-DD>` | #`<编号列表>` |

跨项目待办：[backlog.md](backlog.md)

## 跨仓协议（涉及协议的任务开工前必读）

> 协议正式文档在 [`../docs/protocols/`](../docs/protocols/README.md)。单仓项目本表通常为空。

| 协议 | 版本 | 实现/同步副本 |
|------|------|---------------|

## 每个项目的文件

| 文件 | 性质 | 说明 |
|------|------|------|
| `overview.md` | **可覆写快照** | 定位/模块地图/关键约定/当前局面/最后核准时间 + **knowledge 索引表（唯一入口，无索引视为不存在）**。任务收尾必须刷新 |
| `knowledge/<topic>.md` | 主题文档 | 一主题一文，文头标来源任务与最后核准日期；必须登记进 overview 索引表 |
| `discoveries.md` | 收件箱 | 零散发现先记这里；同主题攒到约 30 行拆成 `knowledge/<topic>.md` 并从收件箱移除 |
| `changelog.md` | 追加 | 控制仓侧变更日志。与业务仓侧**双写**，时机不同：业务仓侧随每次变更完成即追加；控制仓侧任务收尾集中追加 |
| `architecture.md` | 追加 | 架构决策记录 |

新接入一个仓时，从 [templates/project/](../templates/project/) 复制上述文件骨架到 `projects/<repo>/`，并在 `config/projects/<repo>.md` 建注册表（模板 [templates/project-registry.md](../templates/project-registry.md)）。

## 规则

1. **任务收尾**：追加本仓 `changelog.md`、刷新 `overview.md`（含最后核准时间与 knowledge 索引表）、发现进 `discoveries.md`（到阈值拆主题）、遗留项进 [backlog.md](backlog.md)、更新本页看板行
2. **启动新任务前**：看板 → 目标项目 `overview.md` → 按索引表按需读 `knowledge/`（不全量读）→ 扫 [backlog.md](backlog.md)
3. 追加类文件只追加不覆盖；`overview.md` 整体覆写
4. 敏感信息不得写入
```

- [ ] **步骤 4：验证**

```bash
test -f projects/README.md && test -f projects/backlog.md && ! test -f backlog.md && echo OK
```
预期：`OK`

- [ ] **步骤 5：Commit**

```bash
git add projects/ && git commit -m "feat: 新建 projects/ 项目层——全局看板 + backlog 迁移（补项目列与撞号顺延规则）"
```

---

### 任务 2：每仓知识库模板 templates/project/（并删根 knowledge/ 与 changelog.md）

**文件：**
- 创建：`templates/project/overview.md`、`templates/project/discoveries.md`、`templates/project/changelog.md`、`templates/project/architecture.md`
- 删除：`knowledge/`（含 overview.md、discoveries.md、topics/.gitkeep）、`changelog.md`

- [ ] **步骤 1：迁移三个现有文件**

```bash
mkdir -p templates/project
git mv knowledge/overview.md templates/project/overview.md
git mv knowledge/discoveries.md templates/project/discoveries.md
git mv changelog.md templates/project/changelog.md
git rm knowledge/topics/.gitkeep
```

- [ ] **步骤 2：改写 templates/project/overview.md**（`topics/` → `knowledge/`、接入措辞改注册纳管；整体覆写为下述内容）

```markdown
# 项目快照（overview）

> **可覆写快照**——本文件**整体覆写更新**（区别于只追加文件）；任务收尾必刷新。
> 最后核准：<日期>
> 业务仓注册纳管时，由 Agent 探索该仓后初始化本文件（填掉全部 `<...>` 占位）。
> 零散发现不写这里——先进 [discoveries.md](discoveries.md) 收件箱，攒成主题后拆入 `knowledge/` 并登记进下方索引表。

## 定位

<一句话说明本项目是什么、解决什么问题、服务谁>

## 模块地图

<顶层目录/模块 → 职责的映射表或列表，指出核心入口文件>

## 关键约定

<接口约定、命名约定、目录约定、依赖与构建约定等——只写「不知道会踩坑」的>

## 当前局面

<最近完成了什么、在途什么、下一步是什么（可引用 task#NNNN 与 backlog 编号）>

## knowledge 索引表

> `knowledge/` 主题文档的**唯一入口——无索引视为不存在**。新增/更新主题文档必须同步登记本表。

| 主题文档 | 内容 | 最后核准 |
|----------|------|----------|
<!-- 格式示例：| [knowledge/auth.md](knowledge/auth.md) | 鉴权体系：token 获取与刷新约定 | 2026-08-03 | -->
```

- [ ] **步骤 3：改写 templates/project/discoveries.md**（`topics/<topic>.md` → `knowledge/<topic>.md`）

用 Edit 把文中两处 `topics/<topic>.md` 改为 `knowledge/<topic>.md`，把「topics 索引表」改为「knowledge 索引表」。其余内容不动。

- [ ] **步骤 4：改写 templates/project/changelog.md**（补双写口径；整体覆写为下述内容）

```markdown
# changelog（追加型变更日志·控制仓侧）

**规则**：只追加不覆盖；任务收尾时追加一条（未开任务目录的琐碎改动，在变更摘要中附豁免声明备注，见 AGENTS.md §2 豁免条款）。与业务仓侧**双写**：业务仓侧（CHANGELOG.md 或该仓自身约定）随每次变更完成即追加，本文件任务收尾集中追加。

| 日期 | commit | 变更摘要 | 关联任务 |
|------|--------|----------|----------|
```

- [ ] **步骤 5：新建 templates/project/architecture.md**（全文如下）

```markdown
# 架构决策（追加型）

> 只追加不覆盖；本仓架构发生变化时追加一条。

<!-- 格式示例：
## 2026-08-05 事件存储从进程内队列改为 Redis Stream（来源：task#0012）

- 背景：多副本部署后进程内队列丢消息
- 决策：改用 Redis Stream，消费组按副本命名
- 影响：新增 REDIS_URL 配置；旧队列代码下线
-->
```

- [ ] **步骤 6：验证**

```bash
ls templates/project/ && ! test -d knowledge && ! test -f changelog.md && echo OK
grep -c 'knowledge/' templates/project/overview.md
```
预期：列出四个文件 + `OK`；第二条命令 ≥2（索引表引用已切换）。

- [ ] **步骤 7：Commit**

```bash
git add -A templates/project/ knowledge changelog.md 2>/dev/null; git commit -m "feat: 每仓知识库模板 templates/project/ 四件套（根 knowledge/ 与 changelog.md 迁入）"
```

---

### 任务 3：注册表模板 + config/ + .gitignore（并删根 project.md）

**文件：**
- 创建：`templates/project-registry.md`、`config/projects/.gitkeep`、`.gitignore`
- 删除：`project.md`

- [ ] **步骤 1：新建 templates/project-registry.md**（全文如下；字段=原 project.md 全部内容 + 新增分组）

```markdown
# 每仓注册表（config/projects/<repo>.md）

> 业务仓注册纳管时，复制本模板到 `config/projects/<repo>.md` 并填写。**实例不入库**（gitignore）——本机路径各人不同；本模板入库。`<...>` 为待填占位，初始化完成后不得残留。
> 本文件是 AGENTS.md 之下的**仓级事实源**（事实源优先级见 AGENTS.md §6）。

## 基本信息

| 项 | 值 |
|----|----|
| 仓名 | `<repo>` |
| 本机路径 | `<仓库根目录绝对路径>` |
| 类型 | `<如：Web 服务 / CLI 工具 / 库 / 前端应用>` |
| 技术栈 | `<如：Python 3.12 + FastAPI + PostgreSQL>` |

## 分支模型参数（AGENTS.md 硬红线 1 的参数来源）

| 参数 | 值 | 说明 |
|------|----|------|
| 主干分支 | `<main / master>` | 禁止直接 commit/push |
| 集成分支 | `<feat/develop；单人小仓可填主干>` | 只做基线汇聚，禁止在其上开发 |
| 工作分支模式 | `<feat/dev-<user>；或 task/NNNN-<简称>>` | 日常开发与 CR/部署的唯一基准 |
| worktree 根目录 | `<如 ../.worktrees/>` | 业务仓写码一律 worktree 隔离 |

## clean 预检排除项（AGENTS.md 硬红线 2 的参数来源）

```text
<如：':!tmp' ':!logs' ':!.qoder'；无则留空>
```

## 自测命令

本仓完成任务后**必须执行**的验证命令（记入任务 README 自测证据）：

```bash
# 示例格式（按本仓实际替换）：
# uv run pytest -q          # 单元测试
# npm run lint && npm test  # lint + 测试
<自测命令>
```

## 变更规约表

> 架构师开发前**逐条核对**；触发的条目必须**随本次变更一并完成**并记入任务 README 变更记录。
> 第 1-2 行为通用示例，接入时按本仓实际替换或删除。

| # | 触发条件 | 必须伴随的变更 | 验证方式 |
|---|----------|----------------|----------|
| 1 | 新增/修改数据库表结构 | `migrations/` 新增迁移文件 | 文件存在且与 ORM 对齐 |
| 2 | 新增/修改对外 API | 更新 API 文档 | 文档覆盖所有新端点 |
| 3 | `<触发条件>` | `<必须伴随的变更>` | `<验证方式>` |

## 验证性动作清单（需求对齐即授权的自主动作）

> 需求对齐后无需逐次确认即可执行的**验证性**动作及其命令；不在此清单的破坏性/共享状态动作仍受硬红线 5 约束。没有就留空。

| 动作 | 命令 | 说明 |
|------|------|------|
| `<如：预发部署>` | `<命令>` | `<说明>` |
| `<如：内部包发版>` | `<命令>` | `<说明>` |

## 关键约束

从本仓自身规范（`AGENTS.md`/`CLAUDE.md`/贡献指南等）提取的硬性行为限制：

- `<约束一：如某目录禁止直接修改>`
- `<约束二>`
```

- [ ] **步骤 2：建 config/ 目录与 .gitignore、删根 project.md**

```bash
mkdir -p config/projects && touch config/projects/.gitkeep
printf '.DS_Store\nconfig/projects/*\n!config/projects/.gitkeep\n' > .gitignore
git rm project.md
```

- [ ] **步骤 3：验证**

```bash
test -f templates/project-registry.md && ! test -f project.md && echo OK
echo "config/projects/test.md" | git check-ignore --stdin && echo IGNORED
```
预期：`OK`；`IGNORED`（注册表实例被忽略）。

- [ ] **步骤 4：Commit**

```bash
git add .gitignore config/projects/.gitkeep templates/project-registry.md project.md 2>/dev/null; git commit -m "feat: 每仓注册表模板 + config/projects/ 实例目录（不入库）——根 project.md 迁入并扩展分支模型/预检/授权动作参数"
```

---

### 任务 4：docs/protocols/ 跨仓协议文档位

**文件：**
- 创建：`docs/protocols/README.md`

- [ ] **步骤 1：新建 docs/protocols/README.md**（全文如下）

```markdown
# 跨仓协议文档

跨仓契约（事件协议、接口约定、数据格式）的正式文档位。规则：

- **一协议一文**，文头标版本号与「实现/同步副本」位置（业务仓内的副本路径）
- 涉及协议的任务**开工前必读**对应协议文档
- 协议变更时更新版本号、同步各副本，并登记进 [projects/README.md](../../projects/README.md)「跨仓协议」索引表——无索引视为不存在
- 契约按**加法演进**：新增可选字段而非修改既有语义（AGENTS.md 六判据④）

单仓项目（N=1）本目录通常为空——出现「一份契约、两处实现」时才需要它。
```

- [ ] **步骤 2：验证 + Commit**

```bash
test -f docs/protocols/README.md && echo OK
git add docs/protocols/ && git commit -m "feat: docs/protocols/ 跨仓协议文档位"
```

---

### 任务 5：AGENTS.md 组 A——接入模型与事实源

**文件：**
- 修改：`AGENTS.md:3-5`（开头）、`AGENTS.md:129-136`（§6）

- [ ] **步骤 1：改开头事实源清单与接入方式**

L3 中 `（任务文档、知识库、backlog、changelog、project.md）` 改为：
`（任务文档、分仓知识库、看板、backlog、changelog、注册表）`

L5 整行 blockquote 替换为：

```markdown
> 接入方式：本 harness 是**独立控制仓**——自身独立成仓（或独立目录），业务仓通过 `config/projects/<repo>.md` 注册后纳管；单仓项目注册一个仓即可（N=1 退化情形，结构不变）。接入说明见 [README.md](README.md)。
```

- [ ] **步骤 2：改 §6 事实源优先级**

原第 3 条 `3. \`knowledge/\`（项目知识库）+ [project.md](project.md)（项目注册表：技术栈、自测命令、变更规约）` 替换为：

```markdown
3. `projects/<repo>/`（项目知识库）+ `config/projects/<repo>.md`（注册表：本机路径、技术栈、自测命令、变更规约、分支模型参数）
```

原第 4 条中「目标项目自身的规范文件」改为「业务仓自身的规范文件」。

- [ ] **步骤 3：验证 + Commit**

```bash
grep -n 'project\.md' AGENTS.md | grep -v 'config/projects' | grep -v 'README' || echo CLEAN
git add AGENTS.md && git commit -m "docs(AGENTS): 接入模型改独立控制仓 + 事实源优先级按仓化"
```
预期：`CLEAN`（AGENTS.md 中除 config/projects 引用外无孤立 project.md）。

---

### 任务 6：AGENTS.md 组 B——硬红线 1 分支模型抽象化

**文件：**
- 修改：`AGENTS.md:123`（§5 硬红线 1）

- [ ] **步骤 1：整条替换硬红线 1**

原文（L123，「**分支纪律**：禁止在 main/master 上直接 commit/push…」整条）替换为：

```markdown
1. **分支纪律（抽象规则，参数在各仓注册表）**：
   - 禁止在任何仓的主干分支上直接 commit/push。
   - 每仓注册表声明三个参数：**主干分支** / **集成分支** / **工作分支模式**。推荐约定：工作分支 `feat/dev-<user>` → 集成分支 `feat/develop`；单人小仓可声明集成分支=主干。
   - **集成分支只做基线汇聚**：禁止在其上开发或 commit，本地也不长期停在它上面；CR/部署类验证动作只基于工作分支，**禁止用集成分支触发**（多人改动汇聚，失败无法归因）。
   - 验收收口时「工作分支 → 集成分支」的合并 + 推送是需求对齐即授权的自主动作（merge 不 rebase）；成果不得长期只停在工作分支——否则他人从集成分支拉不到、基线断裂。**面向主干的合入始终人工**（集成分支=主干的单人小仓除外：验收后合入主干属收口动作）。
   - **业务仓写码一律在 worktree 内进行**（`git worktree add <worktree根>/<repo>-<NNNN> -b <工作分支>`），主仓目录保持 clean；worktree 验证期间可保留复用，最终验收后删除（删除属破坏性操作，须用户确认，见红线 5）。
```

- [ ] **步骤 2：验证 + Commit**

```bash
grep -n '集成分支只做基线汇聚' AGENTS.md && echo OK
git add AGENTS.md && git commit -m "docs(AGENTS): 硬红线1 分支模型抽象化——参数下沉注册表 + worktree 硬要求"
```

---

### 任务 7：AGENTS.md 组 C——新增硬红线 6-8 多人协作纪律

**文件：**
- 修改：`AGENTS.md` §5（在现红线 5 之后追加）

- [ ] **步骤 1：在硬红线 5 之后追加三条**

```markdown
6. **基线同步与任务去重（开工前）**：开新任务前——控制仓 `git fetch origin && git merge origin/<控制仓主干>` 并入自己的工作分支；业务仓 `git fetch origin && git merge origin/<集成分支>` 并入工作分支；冲突先解决再动工。同步后扫 `tasks/`、`projects/backlog.md`、看板在途分支，核对新任务与他人在途/历史任务无重叠；发现重复先停下协调（拆分/合并/接手/改号）再开工，不得并行推进重复任务。单人使用时退化为「开工前拉最新」一步。
7. **并行任务防冲突**：任务进行中只写自己的 `tasks/<task-id>/` 目录；控制仓共享文件（看板、overview、backlog、changelog）只在任务收尾时集中更新一次，更新前先 `git status` 检查有无其他会话的未提交改动——有则只提交自己的文件，不得提交、覆盖或还原他人改动；暂存区有他人内容时必须用 `git commit -- <自己的路径>` 限定 pathspec 提交，禁止裸 `git commit`。业务仓侧靠 worktree 物理隔离，同文件冲突留到合并阶段解决。**基线前进合并**：任一任务分支合入某仓集成分支后，该仓其他在途任务分支应在下次继续开发前（最迟进入下一轮验证前）在各自 worktree 内 `git merge <集成分支>`；冲突当场解决，不用 rebase。
8. **跨会话主动协同**：发现他会话在途改动、编号/资源撞车、或自己的操作影响他人时，主动留痕协调——事实记入自己任务 README；跨会话需对方知晓的信息（占号声明、冲突说明、交接事项）随时追加到 `projects/backlog.md` 对应条目；新增问题条目同样不受「收尾集中更新」限制——均用 pathspec 限定提交。backlog 即跨会话公告板，另一会话收尾时必读。**编号撞号顺延**：任务编号与 backlog 编号均取现有最大 +1；并行会话撞号，后提交者顺延改号。
```

- [ ] **步骤 2：验证 + Commit**

```bash
grep -c '^[678]\. \*\*' AGENTS.md
git add AGENTS.md && git commit -m "docs(AGENTS): 新增硬红线6-8 多人协作纪律——基线同步/并行防冲突/跨会话协同"
```
预期：grep 计数 3。

---

### 任务 8：AGENTS.md 组 D+E——预检按仓 + 多仓任务口径 + 全文路径更新

**文件：**
- 修改：`AGENTS.md:124`（硬红线 2）、`AGENTS.md:46`（豁免条款）、`AGENTS.md:50`（§2.1）、`AGENTS.md:65`（排查型会话）、`AGENTS.md:84-93`（§2.3 收口）、`AGENTS.md:95-104`（§3 会话铁律）

- [ ] **步骤 1：替换硬红线 2**

原文（L124「**clean 预检**：开工前工作区必须 clean…」整条）替换为：

```markdown
2. **clean 预检（按仓执行）**：预检对象是本次任务要写的业务仓——写码前在该仓执行 `git status --porcelain -- . <排除项>`（排除项由该仓注册表声明，无声明则不带）。**有任何输出即停止并报告用户**，禁止 stash/reset/clean 覆盖用户在途改动。涉及多仓的任务逐仓预检。
```

- [ ] **步骤 2：§2.1 追加多仓任务口径**（任务 ID 段落之后、六件套表之前插入）

```markdown
**多仓任务口径**：一个任务可涉及多个业务仓。`plan.md` 必须列「涉及仓库/分支」（逐仓：工作分支、worktree 路径、基线）；任务 README 变更记录表按仓分行（仓库列为首列）。跨仓同源逻辑无法共享的，至少两侧注释互指 + 自查节声明「同源双份，改动需双改」（六判据③口径）。
```

- [ ] **步骤 3：路径更新（五处）**

| 位置 | 原文关键片段 | 改为 |
|------|--------------|------|
| L46 豁免条款 | `写入 changelog 条目备注` | `写入对应仓 projects/<repo>/changelog.md 条目备注` |
| L65 排查型会话 | `[knowledge/discoveries.md](knowledge/discoveries.md) 或新增 [backlog.md](backlog.md)` | `` `projects/<repo>/discoveries.md` 或新增 [projects/backlog.md](projects/backlog.md) `` |
| §2.3 收口清单（L88-93） | 整个列表 | 见下方替换块 |
| §3 铁律 1（L99） | `读 [knowledge/overview.md](knowledge/overview.md) 恢复项目认知 → 按其索引表**按需**读 \`knowledge/topics/\`（不全量读）→ 扫 [backlog.md](backlog.md)` | `读 [projects/README.md](projects/README.md) 全局看板 → 目标项目 \`projects/<repo>/overview.md\` 恢复认知 → 按其索引表**按需**读 \`projects/<repo>/knowledge/\`（不全量读）→ 扫 [projects/backlog.md](projects/backlog.md)` |
| §3 铁律 2（L100） | `零散发现写 \`knowledge/discoveries.md\`` | `零散发现写 \`projects/<repo>/discoveries.md\`` |

§2.3 收口清单替换块：

```markdown
- 任务 README 补验收记录
- 刷新 `projects/<repo>/overview.md`（快照 + knowledge 索引表 + 最后核准时间），更新 [projects/README.md](projects/README.md) 看板行
- 零散发现归入 `projects/<repo>/discoveries.md`，同主题攒到约 30 行拆成 `projects/<repo>/knowledge/<topic>.md` 并登记索引
- changelog **双写**：业务仓侧随每次变更完成即追加（该仓自身约定）；控制仓侧 `projects/<repo>/changelog.md` 收尾集中追加
- 遗留项登记 [projects/backlog.md](projects/backlog.md)（新条目取最大编号 +1）
- 任务分支合并回集成分支并推送（分支模型参数见各仓注册表，硬红线 1；删除分支须用户确认，硬红线 5）
```

- [ ] **步骤 4：验证 + Commit**

```bash
grep -n 'knowledge/overview\|knowledge/topics\|knowledge/discoveries\|](backlog\.md)\|](changelog\.md)' AGENTS.md || echo CLEAN
git add AGENTS.md && git commit -m "docs(AGENTS): 预检按仓 + 多仓任务口径 + 收口/铁律路径按 projects/ 结构更新"
```
预期：`CLEAN`。

---

### 任务 9：templates/task/ 两处适配

**文件：**
- 修改：`templates/task/plan.md:21-25`（§4）、`templates/task/README.md:40-44`（§4 变更记录）与 `:69-78`（§7 收尾清单）

- [ ] **步骤 1：plan.md §4「涉及分支」改「涉及仓库/分支」**

原 §4 整节替换为：

```markdown
## 4. 涉及仓库/分支

> 多仓任务逐仓一行（AGENTS.md §2.1 多仓任务口径）；分支模型参数见各仓注册表 `config/projects/<repo>.md`。

| 仓库 | 工作分支 | worktree 路径 | 基线 |
|------|----------|---------------|------|
| <repo> | `<工作分支名>` | `<worktree根>/<repo>-<NNNN>` | <基线 commit / 同步时间> |
```

- [ ] **步骤 2：任务 README 模板变更记录表加仓库列**

原表头 `| 分支 | commit | 改动文件 |` 及占位行替换为：

```markdown
| 仓库 | 分支 | commit | 改动文件 |
|------|------|--------|----------|
| <repo> | <分支名> | <sha> | <路径列表> |
```

- [ ] **步骤 3：任务 README 模板收尾清单更新**（§7 复选项替换为）

```markdown
  - [ ] changelog 已双写（业务仓侧随变更已追加；控制仓侧 projects/<repo>/changelog.md 已追加）
  - [ ] projects/<repo>/overview.md 已刷新，projects/README.md 看板行已更新
  - [ ] 遗留项已进 projects/backlog.md
  - [ ] 零散发现已归入 projects/<repo>/discoveries.md（同主题攒到约 30 行的已拆 knowledge/<topic>.md 并登记索引）
  - [ ] 代码自优化自查已填
  - [ ] 任务分支已合并回集成分支并推送（worktree 删除已确认）
```

- [ ] **步骤 4：验证 + Commit**

```bash
grep -n 'knowledge/discoveries' templates/task/README.md || echo CLEAN
git add templates/task/ && git commit -m "docs(templates): 任务模板多仓适配——涉及仓库/分支表、变更记录仓库列、收尾清单新路径"
```
预期：`CLEAN`。

---

### 任务 10：docs/process + docs/principles 路径一致性修订

**文件：**
- 修改：`docs/process/lifecycle.md`（L20-22、41、51、53、63、96-100、104）、`docs/process/sdd.md`（L87、103）、`docs/process/review-loop.md`（L38）、`docs/principles/quality-gates.md`（L11、97）、`docs/principles/harness-coding.md`（L69）

统一替换规则（相对链接自 `docs/process/`、`docs/principles/` 出发均为 `../../`）：

| 原引用 | 新引用 |
|--------|--------|
| `[knowledge/overview.md](../../knowledge/overview.md)` | `` `projects/<repo>/overview.md` ``（并在启动步骤前加「读 [projects/README.md](../../projects/README.md) 全局看板 → 」） |
| `knowledge/topics/`、`topics 索引表` | `projects/<repo>/knowledge/`、`knowledge 索引表` |
| `[knowledge/discoveries.md](../../knowledge/discoveries.md)`、`knowledge/discoveries.md` | `` `projects/<repo>/discoveries.md` `` |
| `[backlog.md](../../backlog.md)` | `[projects/backlog.md](../../projects/backlog.md)` |
| `[changelog.md](../../changelog.md)`、`changelog.md 追加条目` | `` `projects/<repo>/changelog.md` ``（并注明「业务仓侧随变更即追加，双写」，仅 lifecycle L96 与 quality-gates L97 两处需加注） |
| `[project.md](../../project.md)`、`project.md 变更规约表`、`project.md 自测命令`、`可在 project.md 登记` | `` 注册表 `config/projects/<repo>.md` `` 对应小节 |

- [ ] **步骤 1：逐文件按上表 Edit**（lifecycle.md 8 处、sdd.md 2 处、review-loop.md 1 处、quality-gates.md 2 处、harness-coding.md 1 处）

其中 lifecycle.md L100（任务分支合并）整句改为：

```markdown
6. **任务分支合并**：合并回该仓集成分支并推送（分支模型参数见注册表 `config/projects/<repo>.md`；集成分支=主干的单人小仓，验收即授权合入主干，见硬红线 1）。删除分支属破坏性操作，须用户确认（硬红线 5）
```

lifecycle.md L104（完成判据）中 `overview.md + backlog.md` 改为 `看板 + overview.md + projects/backlog.md`。

- [ ] **步骤 2：验证 + Commit**

```bash
grep -rn 'knowledge/\|project\.md\|](\.\./\.\./backlog\.md)\|](\.\./\.\./changelog\.md)' docs/process docs/principles | grep -v 'projects/' | grep -v 'config/projects' || echo CLEAN
git add docs/process docs/principles && git commit -m "docs: process/principles 路径一致性修订——knowledge/project.md/backlog/changelog 引用切至 projects/ 与注册表"
```
预期：`CLEAN`。

---

### 任务 11：examples/minimal-task/ 适配（单仓 N=1 形态）

**文件：**
- 修改：`examples/minimal-task/README.md`（头部 + L74、125、127）、`examples/minimal-task/verification.md`（L17）

- [ ] **步骤 1：README.md 头部 blockquote 追加一句**

在现有头部引言后追加：

```markdown
> 本示例为**单仓（N=1）形态**：业务仓注册名 `mini-cli`，控制仓 `projects/` 下只有 `mini-cli/` 一个目录——单仓与多仓走同一套结构。
```

- [ ] **步骤 2：三处路径替换**

| 位置 | 原文 | 改为 |
|------|------|------|
| README.md:74 | `经验记入 knowledge/discoveries.md` | `经验记入 projects/mini-cli/discoveries.md` |
| README.md:125 | `overview.md 已刷新` | `projects/mini-cli/overview.md 已刷新，看板行已更新` |
| README.md:127 | `零散发现已归入 knowledge/discoveries.md` | `零散发现已归入 projects/mini-cli/discoveries.md` |
| verification.md:17 | `核对自查表尾与 backlog.md` | `核对自查表尾与 projects/backlog.md` |

- [ ] **步骤 3：验证 + Commit**

```bash
grep -rn 'knowledge/' examples/ || echo CLEAN
git add examples/ && git commit -m "docs(examples): 示例适配单仓 N=1 形态——仓名 mini-cli，路径切至 projects/"
```
预期：`CLEAN`。

---

### 任务 12：README.md 全量重写

**文件：**
- 重写：`README.md`

- [ ] **步骤 1：整体覆写 README.md**（全文如下）

```markdown
# agent-dev-harness

一套纯 Markdown 的通用开发流程 harness——**独立控制仓**形态：harness 自身独立成仓，注册 N 个业务仓后纳管（单仓项目 N=1，同一套结构）。AI Agent 读取 [AGENTS.md](AGENTS.md) 即按「需求对齐 → 计划 → 实现 → 审查 → 修复 → 复审 → 验收 → 知识回流」的闭环流程开发，全程留痕、断点可恢复、多人可并行。

## 流程一图流

```text
需求对齐（plan+acceptance 成文）→ 探索/开发/自测 → 代码自优化自查
→ 审查（规格审查+质量审查）→ 修复 → 复审 → 按 acceptance 验证
   ↳ 不通过：归因 → 修复 → 再验证
→ verification.md → 用户最终验收 → 收口（知识回流+changelog+backlog+看板）
```

各环节硬性要求见 [AGENTS.md](AGENTS.md)（最高执行规约），生命周期详述见 [docs/process/lifecycle.md](docs/process/lifecycle.md)。

## 接入三步

1. **建控制仓**：把本目录作为独立仓库（或独立目录）放在业务仓之外，git 管理并提交一次初始状态。
2. **注册业务仓**：每个要纳管的业务仓——复制 [templates/project-registry.md](templates/project-registry.md) 到 `config/projects/<repo>.md`（不入库），让 Agent 探索该仓后填写（本机路径/技术栈/分支模型参数/自测命令/变更规约表）；同时从 [templates/project/](templates/project/) 复制知识库骨架到 `projects/<repo>/` 并初始化 overview.md，在 [projects/README.md](projects/README.md) 看板加一行。单仓项目注册一个仓即可。
3. **开第一个任务**：按 AGENTS.md §2.1 建 `tasks/0001-YYYYMMDD-<简称>/`，从需求对齐开始走完整流程。

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
│   └── projects/<repo>.md            # 每仓注册表（gitignore 不入库：本机路径/分支模型参数/
│                                     #   预检排除项/自测命令/变更规约表/验证性动作清单）
├── tasks/                            # 任务事实源目录（每任务一个子目录，可跨多仓）
│   └── NNNN-YYYYMMDD-<简称>/         #   ← 从 templates/task/ 复制模板填写
├── templates/
│   ├── task/                         # 任务文档六件套模板（README/spec/plan/acceptance/review/verification）
│   ├── project/                      # 每仓知识库骨架（overview/discoveries/changelog/architecture）
│   └── project-registry.md           # 注册表模板（实例在 config/ 不入库）
├── docs/
│   ├── protocols/                    # 跨仓协议文档（一协议一文；单仓时为空）
│   ├── process/                      # 流程细则：lifecycle / sdd / review-loop
│   └── principles/                   # 原则展开：harness-coding / quality-gates
└── examples/
    └── minimal-task/                 # 填好的最小任务示例（单仓 N=1 形态，仓名 mini-cli）
```

### 四层分工

```text
① 规约层（常量）   AGENTS.md ── 硬性规则：角色/流程/会话铁律/六判据/硬红线（含多人纪律）/事实源
② 参考层（常量）   docs/process + docs/principles + docs/protocols + examples + templates
③ 任务层（流水）   tasks/NNNN-YYYYMMDD-<简称>/ ── 一任务一目录，README 是事实源，可跨多仓
④ 项目层（沉淀）   projects/（看板 + 每仓知识库 + backlog）+ config/projects/（每仓注册表）
```

- ①② 与 ③④ 分离：规约与细则不随项目变化；③④ 是各业务仓自身的内容。升级 harness 只替换 ①②。
- ③→④ 回流是闭环关键：任务收口时探索结论进 `projects/<repo>/`、变更进 changelog（双写）、遗留进 backlog、认知刷进 overview + 看板行。
- ④→③ 恢复是会话铁律：新会话靠 看板 → overview → knowledge（按索引）→ backlog 四步建立上下文，再开新任务。

### 一个任务里的文件读写顺序

```text
启动    读 projects/README.md 看板 → 目标仓 overview.md → knowledge/（按索引）→ projects/backlog.md
        恢复在途任务时另读其 tasks/<task-id>/README.md 恢复锚点
对齐    写 plan.md（含设计考量六判据 + 涉及仓库/分支）+ acceptance.md（含 Z 组）[+ spec.md]
开发    读 config/projects/<repo>.md 注册表（预检排除项/变更规约表/自测命令）
        → 逐仓 clean 预检 → worktree 内开发 → 写 README 任务拆分、自测证据
自查    写 README 代码自优化自查表（对 plan 设计考量的落实复核）
审查    写 review.md（每轮追加，REWORK → 修复 → 复审至 PASS）
验证    写 verification.md（对照 acceptance 逐项，失败轮次不删除）
收口    写 README 验收记录 → 刷新 overview.md + 看板行 → discoveries.md → changelog（双写）
        → projects/backlog.md → 任务分支合并回集成分支并推送
```

### 各文件的写入方式

| 写入方式 | 文件 | 说明 |
|---------|------|------|
| **不动** | AGENTS.md、templates/、docs/、examples/ | 除 harness 自身升级外不修改 |
| **覆写 / 就地更新** | projects/<repo>/overview.md、config/projects/<repo>.md | overview 整体覆写；注册表在仓事实变化时就地改 |
| **只追加** | projects/<repo>/changelog.md、discoveries.md、architecture.md | 历史不改写（discoveries 拆主题时整段迁移除外） |
| **新增或更新** | projects/<repo>/knowledge/ | 一主题一文；更新后刷文头「最后核准」并同步 overview 索引表 |
| **追加 + 状态更新** | projects/backlog.md、projects/README.md 看板行 | backlog 新条目取最大编号 +1（撞号后提交者顺延）；看板行收尾更新 |
| **随做随写** | tasks/<task-id>/ | 任务期间实时更新，保证中断后仅凭 README 可续作 |

## 多人并行使用

多人共用一个控制仓远程时，遵守 AGENTS.md 硬红线 6-8：开工前基线同步 + 任务去重、任务中只写自己的任务目录、共享文件收尾集中更新 + pathspec 限定提交、backlog 作跨会话公告板、编号撞号后提交者顺延。单人使用时这些规则自然不触发。

## 从示例开始

不确定一个任务长什么样？先看 [examples/minimal-task/](examples/minimal-task/)——一个填好的自洽示例（单仓 N=1 形态），展示任务 README、plan、acceptance、verification 如何互相引用、逐步写成。
```

- [ ] **步骤 2：验证 + Commit**

```bash
grep -n '复制进\|project\.md' README.md | grep -v 'config/projects\|registry' || echo CLEAN
git add README.md && git commit -m "docs: README 重写——独立控制仓形态接入说明/目录树/四层分工/读写顺序"
```
预期：`CLEAN`。

---

### 任务 13：终检——全仓陈旧引用清零 + 链接可解析

**文件：** 无新改动（有漏网则回到对应任务补）

- [ ] **步骤 1：陈旧引用全仓扫描**

```bash
grep -rn 'knowledge/topics\|](knowledge/\|](\.\./\.\./knowledge/\|](project\.md)\|](backlog\.md)\|](changelog\.md)' --include='*.md' . | grep -v 'docs/superpowers/' || echo CLEAN
```
预期：`CLEAN`（specs/plans 存档目录除外）。有输出则逐条修复后重跑。

- [ ] **步骤 2：Markdown 相对链接可解析检查**

```bash
for f in $(git ls-files '*.md' | grep -v docs/superpowers); do d=$(dirname "$f"); grep -o '](\([^)#<`]*\.md\)' "$f" | sed 's/](//' | while read t; do [ -f "$d/$t" ] || echo "BROKEN: $f -> $t"; done; done; echo DONE
```
预期：无 `BROKEN:` 行，仅 `DONE`。（模板中含 `<repo>` 占位的链接会被 `<` 过滤跳过，属预期。）

- [ ] **步骤 3：目录结构与设计文档 §3 对照**

```bash
ls AGENTS.md README.md projects/README.md projects/backlog.md config/projects/.gitkeep docs/protocols/README.md templates/project-registry.md templates/project/architecture.md && echo STRUCT-OK
```
预期：`STRUCT-OK`。

- [ ] **步骤 4：Commit（如有修复）**

```bash
git status --porcelain || true
# 有修复时：git add -A && git commit -m "docs: 终检修复陈旧引用/断链"
```

---

## 自检记录（编写计划时已执行）

1. **规格覆盖度**：设计 §3 目录结构 → 任务 1-4、12；§4-A → 任务 5；§4-B → 任务 6；§4-C → 任务 7；§4-D/E → 任务 8；§5 注册表 → 任务 3；§6-2 一致性修订 → 任务 9-11；§6-3 示例 N=1 → 任务 11。无遗漏。
2. **占位符扫描**：计划中所有 `<...>` 均为**交付物模板自身的占位符**（接入方填写用），非计划缺口；每个步骤均给出完整替换内容或精确替换规则。
3. **一致性**：仓名示例统一 `mini-cli`；worktree 命名统一 `<worktree根>/<repo>-<NNNN>`；「knowledge 索引表」措辞在任务 2/8/10/12 一致；相对链接深度已按各文件所在目录核对（docs/ 下为 `../../`，projects/ 下为 `../`）。

## 完成后

本任务自身按 harness 流程收口：填写 `tasks/`（若开了任务目录）或至少在 review/verification 阶段走 requesting-code-review + verification-before-completion；最终分支处置（合并回 master / 保留）由用户在验收时决定。
