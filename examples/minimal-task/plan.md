# task#alice-0001 执行计划

> 本文件是填好的示例，展示 templates/task/plan.md 的用法。题材为虚构的 Python CLI 工具 mytool（仓库、路径、commit sha 和输出均为虚构，但填写口径即真实任务的标准）。
> 任务目录：`tasks/alice-0001-20260803-cli-version-flag/`。需求对齐确认后才开工。

## 1. 目标

给 CLI 工具 mytool 增加 `--version` 参数：执行 `mytool --version` 输出 `mytool 1.2.3`（版本号与安装包版本一致，当前为 1.2.3），以 exit code 0 退出，其余行为不变。包元数据读不到（源码目录未安装运行）时输出 `mytool 0.0.0+dev`，不崩溃。

## 2. 背景与根因（缺陷修复时填写）

本任务是新增功能，不涉及缺陷根因。现状证据：`src/mytool/cli.py` 尚未注册 `--version`，执行该参数时 argparse 返回未识别参数。

## 3. 范围

- 做：在 `src/mytool/cli.py` 的 `build_parser()` 注册 `--version`；版本号从 `importlib.metadata` 读取；在 `tests/test_cli.py` 增加功能、兜底和真实入口测试
- 不做：不改既有 `-v/--verbose`、`-q/--quiet` 语义；不新增 `--build-info` 等其他元信息输出，避免任务膨胀

## 4. 设计考量

> 六项在规划阶段填写。说明式写清「担心什么 → 怎么处理」；不涉及的写明理由。编码中若发现判断有误，当场回改本节，并在任务 README 记录变化。

| # | 判据 | 本次的设计考量 |
|---|------|----------------|
| ① | 简洁性 / 尺寸 | `src/mytool/cli.py` 改前实测 75 行，`tests/test_cli.py` 改前实测 120 行；预计分别增加 13 行和 25 行，不会触及文件 >500 行、函数 >60 行、嵌套 >3 层、参数 ≥5 个阈值。完成后仍以 `wc -l` 实测并对照 size-debt 台账 |
| ② | 散点数 | `--version` 只在 `build_parser()` 注册 1 处；版本读取集中在 `get_version()`，无其他注册点或分支需同步修改 |
| ③ | 重复实现 | 担心版本号在 `pyproject.toml` 与代码中双份维护 → 运行时通过 `importlib.metadata.version("mytool")` 读取包元数据，代码中不硬编码版本字符串 |
| ④ | 扩展性 | 本次只增加一个参数和一个纯函数，不新增抽象层、配置开关或接口；当前同类实例不足 2 个，遵守三次法则，不为假想需求预留设计 |
| ⑤ | 分布式 | 本次不涉及——mytool 是单机 CLI 工具，无服务多副本、共享状态、定时任务、跨副本通知或回调 |
| ⑥ | 高可用性 | 本次不涉及服务高可用架构——本地命令失败可重跑；包元数据读取失败时降级为 `0.0.0+dev`，避免非关键路径导致命令崩溃 |

## 5. 实施步骤

- [x] 探索参数注册点、真实命令入口与版本号权威来源
- [x] 先写失败测试，再实现 `--version` 并运行全量测试
- [x] 完成代码自优化自查，填写触碰文件的 `wc -l` 改前→改后实测
- [x] 推送 `feat/dev-alice`，基于该分支创建 CR/PR 并部署预发
- [x] 在预发部署 stage 为 `SUCCESS` 后按 acceptance.md 验证；失败则归因、修复、重新提交和部署，直至复验通过
- [x] 产出 verification.md，提交用户最终验收

## 6. 涉及仓库、分支与 worktree

> 基线固定为 `origin/feat/develop`。本示例是普通任务，使用个人分支；重大功能才使用专项分支。

| 仓库 | 基线 | 工作分支 | worktree 绝对路径 | 同步时间与结果 |
|------|------|----------|-------------------|----------------|
| mini-cli | `origin/feat/develop@9f8e7d6` | `feat/dev-alice` | `/workspace/.worktrees/mini-cli-0001` | 2026-08-03 09:35；merge 完成，无冲突 |

## 7. 风险与回滚

- 风险：① `--version` 若复用 `-v` 会与既有 `--verbose` 撞车；② 未安装场景读取不到包元数据；③ 只测 parser 会漏掉真实 `main()` 入口的组合冲突
- 回滚方式：revert 本任务对应 commit；改动不涉及数据、协议或数据库迁移
