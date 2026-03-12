# Test Cases

## 关联计划
Feature: prim/maze#123 回归 jemalloc testdata 并修复已知兼容性/边界问题

## Test Case 列表

### TC-001: 回归计划文档覆盖关键 smoke 与兼容路径
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: `dev-log/2026-03-12-jemalloc-testdata-regression-plan.md`
- **前置条件**: 仓库存在正式回归计划文档
- **测试步骤**:
  1. 读取回归计划文档。
  2. 检查关键 smoke 用例是否全部列出。
  3. 检查老版本兼容路径用例是否全部列出。
- **预期结果**: 回归计划完整列出关键 smoke、老版本兼容路径与阶段顺序。
- **验证方式**: 自动化静态检查（`tests/test_issue_123_jemalloc_regression.py`）
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_123_jemalloc_regression.TestIssue123JemallocRegression.test_regression_plan_covers_required_cases
  ```

### TC-002: 必需 jemalloc testdata 资产齐备
- **关联 AC**: AC1, AC2
- **类型**: 边界
- **测试数据**: 关键 smoke、兼容版本、边界场景目录
- **前置条件**: `testdata` 子模块已初始化
- **测试步骤**:
  1. 检查 `cpp/20260210-jemalloc-5-3-0`、`cpp/20260211-jemalloc-5-3-0-multithread`、`cpp/20260306-jemalloc-empty-tcache`、`cpp/20260306-jemalloc-extreme-sizes`。
  2. 检查 `cpp/20260211-jemalloc-4-5-0-multithread`、`cpp/20260210-jemalloc-5-0-0`、`cpp/20260210-jemalloc-5-0-1`。
  3. 确认每个目录存在 tarball 与 `validate.py`。
- **预期结果**: 所有计划回归用例目录、tarball、`validate.py` 均齐备。
- **验证方式**: 自动化静态检查
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_123_jemalloc_regression.TestIssue123JemallocRegression.test_required_jemalloc_testdata_assets_exist
  ```

### TC-003: 关键 smoke 用例顺序回归通过
- **关联 AC**: AC1, AC2, AC3
- **类型**: 正向
- **测试数据**: `cpp/20260210-jemalloc-5-3-0`、`cpp/20260211-jemalloc-5-3-0-multithread`、`cpp/20260306-jemalloc-empty-tcache`
- **前置条件**: 已完成构建，测试目录具备 tarball 与 `validate.py`
- **测试步骤**:
  1. 按顺序执行 `python3 testdata/run_test.py <case>`。
  2. 检查退出码是否为 0。
  3. 检查输出中无 `panic`、`fatal`、`Traceback`。
- **预期结果**: 三个 smoke 用例均通过，且主链路无明显异常。
- **验证方式**: 自动化动态回归
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_123_jemalloc_regression.TestIssue123JemallocRegression.test_smoke_cases_pass_via_run_test
  ```

### TC-004: 老版本兼容路径回归通过
- **关联 AC**: AC1, AC3
- **类型**: 兼容性
- **测试数据**: `cpp/20260211-jemalloc-4-5-0-multithread`、`cpp/20260210-jemalloc-5-0-0`、`cpp/20260210-jemalloc-5-0-1`
- **前置条件**: 已完成构建，测试目录具备 tarball 与 `validate.py`
- **测试步骤**:
  1. 按顺序执行三个兼容版本用例。
  2. 检查退出码是否为 0。
  3. 检查输出中无 `panic`、`fatal`、`Traceback`。
- **预期结果**: 老版本兼容路径均通过，无明显崩溃与假通过。
- **验证方式**: 自动化动态回归
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_123_jemalloc_regression.TestIssue123JemallocRegression.test_compatibility_cases_pass_via_run_test
  ```

### TC-005: 修复结果有正式回归 dev-log 记录
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: `dev-log/` 下的 jemalloc 回归结果文档
- **前置条件**: 回归与修复工作已完成
- **测试步骤**:
  1. 在 `dev-log/` 下查找 jemalloc 回归结果文档。
  2. 检查文档包含失败根因、修复方案、复测结果三个章节。
- **预期结果**: 存在正式结果文档，且内容完整可追溯。
- **验证方式**: 自动化静态检查
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_123_jemalloc_regression.TestIssue123JemallocRegression.test_regression_result_devlog_exists_with_required_sections
  ```

## Notes for Developer

- 本轮测试代码同时覆盖静态约束与动态回归主链路。
- 动态用例严格使用 `python3 testdata/run_test.py <case>`，避免直接调用 `validate.py`。
- 当前仓库若缺少 `20260306-jemalloc-empty-tcache` / `20260306-jemalloc-extreme-sizes` 或缺少正式回归结果文档，测试应失败，这是预期的 TDD 信号。
