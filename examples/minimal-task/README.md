# task#alice-0001 cli-version-flag

> 本文件是填好的示例，展示 templates/task/README.md 的用法。题材为虚构的 Python CLI 工具 mytool；任务目录为 `tasks/alice-0001-20260803-cli-version-flag/`，仓库、路径、commit sha、流水线实例和命令输出均为虚构，但填写口径即真实任务的标准。
> 本示例为单仓（N=1）形态：业务仓注册名为 `mini-cli`。

> **恢复锚点（始终保持最新）**
> - 状态：已验收（2026-08-03）
> - 在途分支：`feat/dev-alice`（成果已合入并推送 `feat/develop`；个人分支保留）
> - worktree：`—`（最终验收后已删除 `/workspace/.worktrees/mini-cli-0001`）
> - 当前子任务：全部完成，任务已关闭
> - 下一步：无

- [执行计划](plan.md)
- [验收方案](acceptance.md)
- [自验报告](verification.md)

## 1. 需求与范围

- 目标：给 CLI 工具 mytool 增加 `--version` 参数；`mytool --version` 输出 `mytool 1.2.3`，版本号来自包元数据，exit code 为 0
- 做：在 `src/mytool/cli.py` 注册参数并读取版本；在 `tests/test_cli.py` 增加功能、兜底和真实入口测试
- 不做：不改既有 `-v/--verbose`、`-q/--quiet` 语义；不增加 `--build-info` 等无现实需求的元信息参数
- 用户确认：2026-08-03 09:30 +08:00，摘要：输出格式固定为「程序名 + 空格 + 版本号」；版本号不得硬编码；未安装场景降级为 `0.0.0+dev`

## 2. 任务拆分

> 状态：⬜ 未开始 / ⏳ 进行中 / ✅ 完成 / ❌ 受阻（附原因）。进展实时更新；每个完成项都在下方补对应总结。

| # | 子任务 | 状态 | 结果 / 备注 |
|---|--------|------|-------------|
| 1 | 需求对齐并完成 plan.md、acceptance.md | ✅ | 09:30 确认 |
| 2 | 探索参数注册点、真实入口与版本来源 | ✅ | 结论见 §3 |
| 3 | 先写失败测试，再实现 `--version` | ✅ | a1b2c3d；全量 25 passed |
| 4 | 完成代码自优化自查与尺寸实测 | ✅ | 发现并修复版本号重复维护，e4f5a6b |
| 5 | 基于个人分支完成 CR/PR 与预发部署 | ✅ | `cr-1042`；`deploy-3016` 部署 stage `SUCCESS` |
| 6 | 按 acceptance.md 验证并处理失败回环 | ✅ | A2 首验失败；c7d8e9f；`deploy-3017` 复验通过 |
| 7 | 产出 verification.md 并提交用户最终验收 | ✅ | 10 项全部通过，17:30 验收 |
| 8 | 知识回流并合入 `feat/develop` | ✅ | merge commit m3rg3d1；worktree 已删除 |

### 子任务总结

#### 1. 需求对齐

- 做了什么：与用户对齐目标、范围和验收口径，生成 [plan.md](plan.md) 与 [acceptance.md](acceptance.md)
- 关键产出：明确输出格式、版本单一来源、未安装兜底、预发验证和 Z1-Z4 通过标准
- 问题与解决：无

#### 2. 探索

- 做了什么：检查 `src/mytool/cli.py`、`tests/test_cli.py` 与 `pyproject.toml`
- 关键产出：确认 `build_parser()` 是唯一参数注册点，`main()` 会追加输出选项；版本权威来源是包元数据
- 问题与解决：发现 `-v` 已被 `--verbose` 占用，计划中登记为风险

#### 3. 测试与实现

- 做了什么：先增加 `test_version_output` 并确认红灯，再在 `build_parser()` 注册 `--version`，运行新增测试和全量测试
- 关键产出：a1b2c3d；失败测试 exit 1，初版实现后全量 25 passed
- 问题与解决：初版把 `1.2.3` 硬编码在代码中，后续自查时按判据③修复

#### 4. 代码自优化自查

- 做了什么：实测全部触碰文件行数并逐项检查六判据；发现版本号与 `pyproject.toml` 重复维护
- 关键产出：e4f5a6b 改用 `importlib.metadata.version("mytool")`，增加 `PackageNotFoundError` 兜底与对应测试；全量 26 passed
- 问题与解决：判据③初判有误，发现后立即回改自查结论并完成修复

#### 5. CR/PR 与预发部署

- 做了什么：推送 `feat/dev-alice`，以该分支创建 `cr-1042` 并触发预发流水线 `deploy-3016`
- 关键产出：「预发部署」stage=`SUCCESS`；流水线总状态仍为 `WAITING`，按规则不等待总状态终态
- 问题与解决：无；未使用 `feat/develop` 作为 CR/PR 或预发来源

#### 6. 验证失败回环

- 做了什么：部署 stage 成功后按 acceptance.md 验证。A2 首验暴露 `-v` 冲突；归因并修复后重新提交、部署和全量复验
- 关键产出：c7d8e9f 移除 `--version` 的短选项并增加真实 `main()` 入口测试；`deploy-3017` 部署 stage=`SUCCESS`；10 项复验通过
- 问题与解决：完整失败证据、归因、重新提交、重新部署和复验记录见 [verification.md](verification.md) 第 1 轮

#### 7. 自验与用户最终验收

- 做了什么：产出 verification.md，对照 acceptance.md 逐项提供命令、cwd、exit code 和证据；提交用户最终验收
- 关键产出：A 组 4 项、B 组 2 项、Z 组 4 项全部通过；用户于 17:30 确认
- 问题与解决：无

#### 8. 收口

- 做了什么：完成 changelog 双写、overview 与看板刷新、discoveries 回流；用户最终验收后将 `feat/dev-alice` 合入并推送 `feat/develop`，删除本任务 worktree
- 关键产出：merge commit m3rg3d1；`feat/develop` 已推送；未合入或推送 `main`/`master`
- 问题与解决：本任务没有临时任务分支；长期个人分支 `feat/dev-alice` 保留

## 3. 探索结论

- `src/mytool/cli.py` 改前 75 行；`build_parser()` 是唯一参数注册点，`main()` 会调用 `add_output_options()` 注册 `-v/--verbose` 与 `-q/--quiet`
- 版本号权威来源为 `pyproject.toml` 的项目元数据；安装后可通过 `importlib.metadata.version("mytool")` 读取
- 未安装的纯源码环境会抛 `PackageNotFoundError`，需降级为 `0.0.0+dev`
- 改动前基线 `origin/feat/develop@9f8e7d6`：`pytest tests/ -q` 为 24 passed，无存量失败

## 4. 变更记录

| 仓库 | 分支 | commit | 改动文件 |
|------|------|--------|----------|
| mini-cli | `feat/dev-alice` | a1b2c3d（初始实现） | `src/mytool/cli.py`、`tests/test_cli.py` |
| mini-cli | `feat/dev-alice` | e4f5a6b（版本单一来源与兜底） | `src/mytool/cli.py`、`tests/test_cli.py` |
| mini-cli | `feat/dev-alice` | c7d8e9f（移除短选项冲突并补真实入口测试） | `src/mytool/cli.py`、`tests/test_cli.py` |
| mini-cli | `feat/develop` | m3rg3d1（用户最终验收后收口合并） | 合并上述成果 |

## 5. 自测证据

| 命令 | cwd | exit code | 结果摘要 |
|------|-----|-----------|----------|
| `pytest tests/test_cli.py::test_version_output -q` | `/workspace/.worktrees/mini-cli-0001` | 1 | 1 failed；`--version` 未注册，TDD 红灯符合预期 |
| `pytest tests/ -q` | `/workspace/.worktrees/mini-cli-0001` | 0 | 25 passed（a1b2c3d） |
| `pytest tests/ -q` | `/workspace/.worktrees/mini-cli-0001` | 0 | 26 passed（e4f5a6b） |
| `python -m mytool.cli --version` | `/workspace/.worktrees/mini-cli-0001` | 0 | 输出 `mytool 0.0.0+dev`，无 traceback |
| `mytool --version` | 预发环境 | 1 | 首验失败：`conflicting option string: -v`；见 verification.md 第 1 轮 |
| `mytool --version` | 预发环境 | 0 | c7d8e9f 与 `deploy-3017` 后输出 `mytool 1.2.3` |
| `pytest tests/ -q` | `/workspace/.worktrees/mini-cli-0001` | 0 | 27 passed（最终全量） |
| `pytest tests/test_cli.py -v` | `/workspace/.worktrees/mini-cli-0001` | 0 | 8 passed，包含既有选项与新增 3 条用例 |

## 6. 代码自优化自查

> 本节是对 [plan.md](plan.md)「设计考量」的落实复核。六项逐条填写结论与动作；不涉及的写明理由。

### 6.1 尺寸证据

> 行数均由 `wc -l` 实测，并对照 `projects/mini-cli/knowledge/size-debt.md`。

| 仓库 | 文件 | 改前行数 | 改后行数 | 变化 | 阈值 / 是否触线 | size-debt 对照与动作 |
|------|------|----------|----------|------|-----------------|----------------------|
| mini-cli | `src/mytool/cli.py` | 75 | 88 | +13 | 文件未超过 500 行；新增 `get_version()` 12 行，函数未超过 60 行；嵌套 2 层；参数 0 个，不触线 | 台账无该文件；无需登记 |
| mini-cli | `tests/test_cli.py` | 120 | 145 | +25 | 文件未超过 500 行；新增测试函数最长 9 行；嵌套 1 层；参数最多 1 个，不触线 | 台账无该文件；无需登记 |

### 6.2 六判据结论

| # | 判据 | 结论 | 动作 |
|---|------|------|------|
| ① | 尺寸报警 | 2 个触碰文件均有改前→改后实测；文件、函数、嵌套和参数均未触线 | 无需拆分或登记尺寸债 |
| ② | 散点数 | `--version` 只在 `build_parser()` 注册 1 处；版本读取集中在 `get_version()` | 无需动作 |
| ③ | 重复实现 | 曾触线：a1b2c3d 硬编码 `1.2.3`，与包元数据重复维护 | e4f5a6b 改为从 `importlib.metadata` 读取；`grep -rn "1\.2\.3" src/` 无输出 |
| ④ | 扩展性 | 本次只增加参数与纯函数，未新增抽象层、配置开关或接口；没有为假想需求预留代码 | 遵守三次法则，无需动作 |
| ⑤ | 分布式 | 本次不涉及——mytool 是单机 CLI 工具，无多副本、共享状态、定时任务、跨副本通知或回调 | 无需动作 |
| ⑥ | 高可用性 | 本次不涉及服务高可用架构；包元数据读取失败时降级为 `0.0.0+dev`，命令不崩溃 | e4f5a6b 已实现并由 `test_version_fallback_dev` 覆盖 |

未处理的触线项已记 backlog：无；唯一触线项③已在本任务内修复。

## 7. 验收记录

- 用户确认：2026-08-03 17:30 +08:00，范围：acceptance.md 的 A/B/Z 三组共 10 项全部通过，包含 A2 失败轮次及重新提交、部署、复验证据
- 收尾核对：
  - [x] 业务仓与控制仓 changelog 已双写；控制仓条目引用 `task#alice-0001`
  - [x] `projects/mini-cli/overview.md`、看板与 discoveries 已更新；knowledge 索引无新增
  - [x] 本任务无遗留项，backlog 无新增
  - [x] 本次未改变代码目录或模块结构，业务仓 CodeMap 不适用
  - [x] 代码自优化自查及 2 个触碰文件的尺寸实测已填写
  - [x] 用户最终验收后，`feat/dev-alice` 已合入并推送 `feat/develop`
  - [x] 未合入或推送 `main`/`master`；生产合入与生产部署留给人工
  - [x] 本任务 worktree 已删除；无临时任务分支需要删除
