# Test Cases

## 关联计划
Feature: prim/maze#87 重新执行 Python 相关 testdata 并修复失败回归

## Test File

- `tests/test_issue_87_python_regression_matrix.py`：将 issue #87 的 Python 回归范围固化为可执行测试矩阵，覆盖普通模式全量回归、`--py-merge` 专项回归以及代表版本覆盖检查。

## Test Case 列表

### TC-001: Python 全量普通模式回归矩阵
- **关联 AC**: AC1 普通模式下所有 Python testdata 全部通过
- **类型**: 正向 / 回归
- **测试数据**: `testdata/python/` 下全部现有 Python 用例
- **前置条件**:
  - `testdata` 子模块已初始化
  - 每个测试目录下存在自己的 `coredump-*.tar.gz` 与 `validate.py`
- **测试步骤**:
  1. 逐个执行 `ALL_PYTHON_TESTDATA` 中列出的 Python 用例
  2. 每个用例统一通过 `testdata/run_test.py` 触发 Maze 分析与 `validate.py` 校验
- **预期结果**: 所有 Python 用例在普通模式下退出码均为 0，无单个回归遗漏
- **验证方式**: 自动化测试文件 `tests/test_issue_87_python_regression_matrix.py::test_python_testdata_pass_in_normal_mode`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260128-basic
  ```

### TC-002: `--py-merge` 回归矩阵
- **关联 AC**: AC2 涉及 `--py-merge` 的用例全部通过
- **类型**: 正向 / 回归 / 准确性
- **测试数据**:
  - `testdata/python/20260201-class-merge-py27`
  - `testdata/python/20260201-class-merge-py311`
  - `testdata/python/20260201-py-merge`
- **前置条件**:
  - `testdata` 子模块已初始化
  - `run_test.py` 可使用 `--py-merge` 模式
- **测试步骤**:
  1. 对每个 `PY_MERGE_TESTDATA` 用例执行 `python3 testdata/run_test.py --py-merge <test_dir>`
  2. 检查普通模式和 `--py-merge` 模式结果都通过
- **预期结果**: 三个 `--py-merge` 相关用例均通过，且无退出码异常
- **验证方式**: 自动化测试文件 `tests/test_issue_87_python_regression_matrix.py::test_python_py_merge_regressions_pass`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py --py-merge python/20260201-class-merge-py311
  ```

### TC-003: 代表 Python 版本覆盖检查
- **关联 AC**: AC3 至少覆盖 Python 2.7、3.11、3.12 三类代表版本验证
- **类型**: 兼容性
- **测试数据**:
  - `testdata/python/20260129-complex-types-27`
  - `testdata/python/20260129-complex-types-311`
  - `testdata/python/20260129-complex-types-312`
- **前置条件**: `testdata/python/` 目录完整可用
- **测试步骤**:
  1. 检查 issue #87 的回归矩阵中是否显式包含 Python 2.7、3.11、3.12 代表版本目录
  2. 确认这些目录处于普通模式全量回归列表中
- **预期结果**: 三类代表版本均被纳入自动化回归范围
- **验证方式**: 自动化测试文件 `tests/test_issue_87_python_regression_matrix.py::test_python_regression_scope_covers_required_representative_versions`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-312
  ```

### TC-004: Python 回归范围完整性检查
- **关联 AC**: AC4 无修复引入的新回归
- **类型**: 边界 / 回归防逃逸
- **测试数据**: `testdata/python/` 目录清单
- **前置条件**: `testdata` 子模块已初始化
- **测试步骤**:
  1. 扫描 `testdata/python/` 下所有包含 `validate.py` 的目录
  2. 与测试代码中的 `ALL_PYTHON_TESTDATA` 进行一一比对
- **预期结果**: 自动化矩阵与实际 Python testdata 清单完全一致，避免新增或遗漏目录未被回归
- **验证方式**: 自动化测试文件 `tests/test_issue_87_python_regression_matrix.py::test_python_regression_inventory_matches_issue_scope`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260211-comprehensive-types
  ```

## Notes for Developer

- 本次仅新增测试，不修改功能实现代码。
- `test_python_testdata_pass_in_normal_mode` 与 `test_python_py_merge_regressions_pass` 会直接执行真实 `run_test.py`，因此当前存在的 Python 回归会在该测试文件中直接暴露出来，这符合 issue #87 的 TDD 目标。
- `test_python_regression_inventory_matches_issue_scope` 用于防止后续新增 Python testdata 后未同步补充回归矩阵，避免“修了一个版本、漏了另一个版本”的问题。
