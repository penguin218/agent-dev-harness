# task#alice-0001 自验报告

> 本文件是填好的示例，展示 templates/task/verification.md 的用法。题材为虚构的 Python CLI 工具 mytool（命令、输出、sha 和流水线实例均为虚构，填写口径即真实任务的标准）。
> 任务目录：`tasks/alice-0001-20260803-cli-version-flag/`。未执行的验证不得写成通过；失败记录不得删除或改写。

## 1. 验收项逐项映射

| acceptance 条目 | 验证方式 / 命令 | cwd | exit code | 证据 | 结果 |
|-----------------|-----------------|-----|-----------|------|------|
| A1 个人分支预发且部署阶段成功 | 核对 `cr-1042` 源分支；`ci pipeline show deploy-3017 --stages` | 不适用 | 0 | 源分支 `feat/dev-alice`；「预发部署」stage=`SUCCESS`；流水线总状态仍为 `WAITING`，未等待总状态终态 | ✅ 通过 |
| A2 `--version` 输出正确版本并正常退出 | 预发环境执行 `mytool --version; echo $?` | 不适用 | 0 | 第 1 轮失败后，c7d8e9f 复验输出 `mytool 1.2.3` 和 `0` | ✅ 通过（经 1 轮修复） |
| A3 版本号单一来源 | `grep -rn "1\.2\.3" src/`；核对读取路径 | `/workspace/.worktrees/mini-cli-0001` | 1（无匹配，符合预期） | `src/` 无硬编码版本；`get_version()` 调用 `importlib.metadata.version("mytool")` | ✅ 通过 |
| A4 未安装场景降级不崩溃 | `python -m mytool.cli --version; echo $?` | `/workspace/.worktrees/mini-cli-0001` | 0 | 输出 `mytool 0.0.0+dev` 和 `0`，无 traceback | ✅ 通过 |
| B1 全量测试无新增失败 | `pytest tests/ -q` | `/workspace/.worktrees/mini-cli-0001` | 0 | `27 passed in 1.84s`；基线为 24 passed，本次新增 3 条 | ✅ 通过 |
| B2 既有 verbose/quiet 行为不变 | `mytool run --verbose demo.txt`；`pytest tests/test_cli.py -v` | `/workspace/.worktrees/mini-cli-0001` | 0 | verbose 输出与基线一致；`test_cli.py` 8 passed | ✅ 通过 |
| Z1 计划设计考量 | 逐项核对 plan.md 六项与最终实现 | 任务目录 | 不适用 | 六项均已填；⑤⑥写明不涉及及理由；最终实现一致 | ✅ 通过 |
| Z2 README 六项落实 | 核对自查表、`wc -l` 改前→改后表及 size-debt 对照 | 任务目录与业务仓 | 不适用 | 2 个触碰文件均有实测；六项均有结论与动作 | ✅ 通过 |
| Z3 触线项处理 | 核对修复 commit、台账与 backlog | 任务目录与项目知识库 | 不适用 | 唯一触线项③已由 e4f5a6b 修复；无未处理项 | ✅ 通过 |
| Z4 三次法则 | 核对新增抽象、配置开关和接口 | `/workspace/.worktrees/mini-cli-0001` | 不适用 | 未新增抽象层、配置开关或接口 | ✅ 通过 |

## 2. 失败轮次记录

> 每次不通过都按「归因 → 修复 → 重新提交 / 发布 / 部署 → 复验」完整追加；不得删除既有失败记录。

### 第 1 轮（2026-08-03 14:50）

- 失败条目：A2
- 现象与证据：流水线实例 `deploy-3016` 的「预发部署」stage 已为 `SUCCESS`；随后在预发执行 `mytool --version`，报 `argparse.ArgumentError: argument -v/--verbose: conflicting option string: -v`，exit 1。此前本地 `pytest tests/ -q` 为 26 passed
- 归因：① a1b2c3d 将 `--version` 注册为 `-v/--version`，与 `main()` 中既有 `-v/--verbose` 冲突；② 新增测试只调用 `build_parser()`，未走完整 `main()` 入口，导致本地测试未暴露组合冲突
- 修复：c7d8e9f 移除 `--version` 的短选项，仅保留长选项；新增 `test_version_via_main` 覆盖真实入口
- 重新提交：c7d8e9f 已推送到 `feat/dev-alice`，CR/PR `cr-1042` 更新到该 commit
- 重新发布：不适用；本项目无独立软件包发布步骤
- 重新部署：触发实例 `deploy-3017`；「预发部署」stage=`SUCCESS`，流水线总状态 `WAITING` 不作为等待条件
- 复验命令：预发执行 `mytool --version; echo $?`；本地执行 `pytest tests/ -q`
- cwd：预发不适用；本地 `/workspace/.worktrees/mini-cli-0001`
- exit code：均为 0
- 复验证据与结果：预发输出 `mytool 1.2.3` 和 `0`；本地 `27 passed in 1.84s`，✅

## 3. 总结论

- 验收结果：A（4 项）、B（2 项）、Z（4 项）共 10 项全部通过
- 降级项与替代证据：无
- 结论：可提交用户最终验收
