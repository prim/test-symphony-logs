# Test Cases

## 关联计划
Feature: prim/maze#132 根据 `maze.log` 堆栈修复 `CountGoroutineError` 并将 error amount 压到 16 以下

## Test File(s)

- `tests/test_issue_132_count_goroutine_error_regression.py`：覆盖 issue #132 的静态约束检查、正式实施记录检查与目标用例动态回归。

## Test Case 列表

### TC-001: 计划文档覆盖目标用例与阈值约束
- **关联 AC**: AC1, AC3
- **类型**: 正向
- **测试数据**: `dev-log/2026-03-12-fix-count-goroutine-error-less-than-16.md`
- **前置条件**: 计划文档存在
- **测试步骤**:
  1. 读取计划文档。
  2. 检查是否包含目标用例 `python/20260128-basic`。
  3. 检查是否包含 `error amount 小于 16`、禁止新增统一错误接口、禁止隐藏日志等约束。
- **预期结果**: 计划文档明确记录目标用例、阈值和修复约束。
- **验证方式**: `unittest` 静态检查。

### TC-002: CountGoroutineError 统计入口仍保留在主链路中
- **关联 AC**: AC3
- **类型**: 反向
- **测试数据**: `main.go`, `common/logger.go`
- **前置条件**: 源码可读
- **测试步骤**:
  1. 检查 `main.go` 是否仍包含 `defer CountGoroutineError()`。
  2. 检查 `common/logger.go` 是否仍定义 `CountGoroutineError`。
- **预期结果**: 没有通过删除统计入口来掩盖真实问题。
- **验证方式**: `unittest` 静态检查。

### TC-003: 开发实施记录包含 QA 必需字段
- **关联 AC**: AC2, AC6, AC7
- **类型**: 正向
- **测试数据**: `dev-log/*count-goroutine*.md`
- **前置条件**: 开发已提交正式实施记录
- **测试步骤**:
  1. 搜索除计划文档外的 `count-goroutine` 相关 dev-log。
  2. 检查是否包含错误堆栈位置、根因、修复文件、修复前后 error amount 与 `< 16` 结论。
- **预期结果**: 至少存在一份正式实施记录且关键信息完整。
- **验证方式**: `unittest` 静态检查。

### TC-004: 目标 Python 用例 smoke 回归后 maze.log 错误量低于阈值
- **关联 AC**: AC1, AC4, AC5
- **类型**: 正向
- **测试数据**: `testdata/python/20260128-basic`
- **前置条件**: `testdata/run_test.py`、目标测试目录、`validate.py` 与 tarball 均存在
- **测试步骤**:
  1. 执行 `python3 testdata/run_test.py python/20260128-basic`。
  2. 确认输出包含 `Running Maze Analysis`，且没有 `panic` / `fatal` / `Traceback`。
  3. 读取根目录 `maze.log`，统计 `goroutine` 关键字出现次数。
- **预期结果**: 目标用例回归成功，且 `maze.log` 中相关错误堆栈数量 `< 16`。
- **验证方式**: `unittest` 动态检查。
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260128-basic
  ```

## Notes for Developer

- 本轮测试明确禁止通过删除 `CountGoroutineError` 主流程入口来“伪造通过”。
- 若 testdata 基础设施未就绪，动态 smoke test 会跳过，但静态测试仍会因为缺少正式实施记录而失败。
- 正式实施记录建议单独新增到 `dev-log/`，不要覆盖计划文档。
