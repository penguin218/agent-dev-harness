# 项目知识库 — 全局看板

> 新会话第一入口：先读本页看板，再进目标项目的 `<repo>/overview.md`（会话铁律，AGENTS.md §3）。

## 看板

> 一行一仓；任务收尾时更新本仓所在行。单仓项目（N=1）本表只有一行。

| 项目 | 当前焦点 | 在途分支 | 最后核准 | 待办（backlog） |
|------|----------|----------|----------|-----------------|
| `<repo>`（接入时替换本示例行） | `<在做/刚验收的任务一句话（可引 task#<user>-NNNN）>` | `<工作分支（commit 短 sha）>` | `<YYYY-MM-DD>` | `#<编号列表>` |

跨项目待办：[backlog.md](backlog.md)

## 跨仓协议（涉及协议的任务开工前必读）

> 协议正式文档在 [`../docs/protocols/`](../docs/protocols/README.md)。单仓项目本表通常为空。

| 协议 | 版本 | 实现/同步副本 |
|------|------|---------------|

## 每个项目的文件

| 位置 | 性质 | 说明 |
|------|------|------|
| `projects/<repo>/overview.md` | **可覆写快照** | 定位/模块地图/关键约定/当前局面/最后核准时间 + **knowledge 索引表（唯一入口，无索引视为不存在）**。任务收尾必须刷新 |
| `projects/<repo>/knowledge/<topic>.md` | 主题文档 | 一主题一文，文头标来源任务与最后核准日期；必须登记进 overview 索引表 |
| `projects/<repo>/discoveries.md` | 收件箱 | 零散发现先记这里；同主题攒到约 30 行拆成 `knowledge/<topic>.md` 并从收件箱移除 |
| `projects/<repo>/architecture.md` | 追加 | 架构决策记录 |
| `projects/<repo>/changelog.md` | 追加 | 控制仓侧变更日志。与业务仓侧**双写**，时机不同：业务仓侧随每次变更完成即追加；控制仓侧任务收尾集中追加 |
| `projects/backlog.md` | 全局待办池 | 任务遗留项与跨会话协调信息；编号全局递增 |
| 业务仓 `mydocs/codemap/` | CodeMap | 代码结构变化时就地更新；无结构变化时记录不适用理由 |

新接入一个仓时，从 [templates/project/](../templates/project/) 复制项目知识库骨架（含 `knowledge/` 空目录）到 `projects/<repo>/`，并在 `config/projects/<repo>.md` 建注册表（模板 [templates/project-registry.md](../templates/project-registry.md)）。

## 规则

1. **任务收尾**：追加本仓 `changelog.md`、刷新 `overview.md`（含最后核准时间与 knowledge 索引表）、发现进 `discoveries.md`（到阈值拆主题）、遗留项进 [backlog.md](backlog.md)、按代码结构变化更新业务仓 CodeMap，并更新本页看板行
2. **启动新任务前**：看板 → 目标项目 `overview.md` → 按索引表按需读 `knowledge/`（不全量读）→ 扫 [backlog.md](backlog.md)
3. 追加类文件只追加不覆盖；`overview.md` 整体覆写
4. 敏感信息不得写入
