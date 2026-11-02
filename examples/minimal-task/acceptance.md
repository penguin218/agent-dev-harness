# task#alice-0001 验收方案

> 本文件是填好的示例，展示 templates/task/acceptance.md 的用法。题材为虚构的 Python CLI 工具 mytool（命令、输出和 sha 均为虚构，填写口径即真实任务的标准）。
> 任务目录：`tasks/alice-0001-20260803-cli-version-flag/`。功能、回归与 Z 组同权；未验证的事项不得视为通过。

## A 组：功能验收

- [x] **A1** 预发部署来自个人分支且部署阶段成功
  - 验证方式：核对 CR/PR `cr-1042` 的源分支；执行 `ci pipeline show deploy-3017 --stages`
  - 通过标准：源分支为 `feat/dev-alice`；「预发部署」stage 为 `SUCCESS`。不以流水线总状态是否终态作为判据
- [x] **A2** `mytool --version` 在预发环境输出正确版本并正常退出
  - 验证方式：在预发环境执行 `mytool --version; echo $?`
  - 通过标准：stdout 恰为 `mytool 1.2.3`，exit code 0
- [x] **A3** 版本号保持单一来源
  - 验证方式：`grep -rn "1\.2\.3" src/`；核对实现读取路径为 `importlib.metadata.version("mytool")`
  - 通过标准：`src/` 下无硬编码版本字符串（grep 无输出），版本值来自包元数据
- [x] **A4** 未安装（纯源码运行）场景降级不崩溃
  - 验证方式：在未安装 mytool 的干净环境执行 `python -m mytool.cli --version; echo $?`
  - 通过标准：输出 `mytool 0.0.0+dev`，exit code 0，无 traceback

## B 组：回归验收

- [x] **B1** 全量测试通过且不新增失败
  - 验证方式：`pytest tests/ -q`
  - 通过标准：基线 24 条 + 本次新增 3 条 = 27 passed，无新增失败
- [x] **B2** 既有 `-v/--verbose`、`-q/--quiet` 行为不变
  - 验证方式：`mytool run --verbose demo.txt` 与改动前输出对比；执行 `pytest tests/test_cli.py -v`
  - 通过标准：verbose/quiet 输出与基线一致，相关用例通过

## Z 组：代码自优化自查

> ①②③④依据代码事实验收；⑤⑥只需说明做了什么，不要求量化指标、压测或专门并发测试。

- [x] **Z1** `plan.md`「设计考量」六项均已填写，且与最终实现一致
  - 验证方式：逐项核对计划与最终改动；检查开发中变化是否已回写
  - 通过标准：无空项；不涉及项均写明「本次不涉及」+ 理由
- [x] **Z2** README「代码自优化自查」六项均已落实
  - 验证方式：核对六项结论与动作、所有触碰文件的 `wc -l` 改前→改后表及 `projects/mini-cli/knowledge/size-debt.md` 对照
  - 通过标准：无空项；尺寸数据为实测值；琐碎改动使用豁免时已说明理由
- [x] **Z3** 所有触线项均已处理或登记 backlog
  - 验证方式：逐项核对修复 commit、尺寸债台账或 backlog 编号与不在本任务处理的理由
  - 通过标准：每个触线项都有可追溯处置；不存在模糊尺寸结论
- [x] **Z4** 遵守三次法则，未引入过度设计
  - 验证方式：检查本次新增的抽象层、配置开关与接口所对应的现有同类实例
  - 通过标准：未为假想需求预留代码；如有抽象，均对应已存在至少 2 个同类实例

## 通过口径

- A、B、Z 三组共 10 项全部通过，才可提交自验报告并请求用户最终验收
- 允许降级项：无
