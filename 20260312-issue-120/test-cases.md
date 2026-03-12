# Test Cases

## 关联计划
Feature: `prim/maze#120` Python testdata 全量回归存在 7 个失败用例，需按版本兼容路径修复

## 测试文件

- `tests/test_issue_120_python_testdata_regression.py`：Issue #120 的 Python testdata 回归门禁，覆盖已知失败用例、代表版本和全量串行回归

## Test Case 列表

### TC-001: 已知失败用例资产完整性检查
- **关联 AC**: AC1, AC3
- **类型**: 反向 / 错误处理
- **测试数据**: `python/20260129-complex-types`、`python/20260129-complex-types-27`、`python/20260129-complex-types-35`、`python/20260129-complex-types-36`、`python/20260129-complex-types-37`、`python/20260129-complex-types-38`、`python/20260201-class-merge-py27`
- **前置条件**: `testdata` 子模块已初始化
- **测试步骤**:
  1. 检查 `testdata/run_test.py` 是否存在
  2. 逐个检查 7 个已知失败用例目录是否存在
  3. 检查每个用例是否包含 `validate.py` 与 `coredump-*.tar.gz`
- **预期结果**: 所有已知失败用例均保持可运行，回归覆盖面未被意外删减
- **验证方式**: 自动化测试 `TestIssue120PythonTestdataRegression.test_known_regression_cases_keep_required_assets`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types
  ```

### TC-002: 代表 Python 版本回归通过
- **关联 AC**: AC2, AC3
- **类型**: 兼容性 / 正向
- **测试数据**: `python/20260129-complex-types-27`、`python/20260129-complex-types-311`、`python/20260129-complex-types-312`
- **前置条件**: `testdata` 子模块已初始化，Maze 可运行
- **测试步骤**:
  1. 依次执行 py27、py311、py312 三个代表版本用例
  2. 检查每个用例的 `run_test.py` 返回码
- **预期结果**: 三个代表版本用例全部通过，说明修复没有破坏主版本兼容面
- **验证方式**: 自动化测试 `TestIssue120PythonTestdataRegression.test_representative_python_versions_pass_regression_gate`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-27
  ```

### TC-003: 7 个已知失败回归全部转绿
- **关联 AC**: AC1, AC3, AC4
- **类型**: 正向 / 回归
- **测试数据**: 同 TC-001 的 7 个失败用例
- **前置条件**: 修复代码已合入当前工作区
- **测试步骤**:
  1. 使用 `python3 testdata/run_test.py <case>` 逐个执行 7 个失败用例
  2. 观察每个用例是否全部返回 0
- **预期结果**:
  - `PersonClass` 识别恢复正常
  - `pymempool_objects` 不再低于阈值
  - `deque=301` 的偏差问题被解释并修复到断言可通过
  - `class-merge-py27` 的 inner/outer dict 能被正确识别
- **验证方式**: 自动化测试 `TestIssue120PythonTestdataRegression.test_known_failed_python_regressions_all_pass_after_fix`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260201-class-merge-py27
  ```

### TC-004: Python testdata 全量串行回归通过
- **关联 AC**: AC1, AC2, AC3, AC4
- **类型**: 正向 / 回归 / 边界
- **测试数据**: 当前 16 个可运行 Python 用例全量集合
- **前置条件**: `testdata/python` 当前可运行集合完整，Maze 可运行
- **测试步骤**:
  1. 按固定列表顺序串行执行全部 16 个 Python 用例
  2. 不并行执行多个用例
  3. 检查所有用例返回码均为 0
- **预期结果**: Python testdata 全量通过，且修复未引入新增回归
- **验证方式**: 自动化测试 `TestIssue120PythonTestdataRegression.test_full_python_testdata_suite_passes_sequentially`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260211-comprehensive-types
  ```

## Notes for Developer

- 新增测试严格要求通过 `testdata/run_test.py` 进入统一验收链路，不能以直接调用 `validate.py` 的方式规避真实问题。
- 代表版本门禁固定为 `py27`、`py311`、`py312`，用于覆盖旧版本兼容路径与当前主线版本。
- 全量回归列表按 2026-03-12 已知可运行集合固化；若后续新增 Python testdata，应同步扩充该列表与文档。
