# Test Cases

## 关联计划
Feature: 保持功能不变，重构 Golang `python` package

## Test Case 列表

### TC-001: Python 综合对象图基线回归
- **关联 AC**: AC1, AC2, AC5
- **类型**: 正向
- **测试数据**: `testdata/python/20260307-refactor-comprehensive-py311`
- **前置条件**:
  - 当前仓库根目录可执行 `python3`
  - 已存在 `./maze` 可执行入口
  - 使用 `cmd/maze-gen-coredump.py` 生成本目录 tarball
- **测试步骤**:
  1. 运行测试数据生成脚本，创建综合 Python 对象图。
  2. 使用 `cmd/maze-gen-coredump.py` 生成该测试目录下的 coredump tar.gz。
  3. 执行统一验收命令运行 Maze 分析。
  4. 检查结果中是否包含核心对象类型与合理数量。
- **预期结果**:
  - Maze 成功完成 Python 主流程分析。
  - 结果中能识别综合测试中的关键对象类型。
  - `summary.total_objects` 大于 0，且无结果结构损坏。
- **验证方式**: 自动化测试命令 + `validate.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260307-refactor-comprehensive-py311
  ```

### TC-002: `--py-merge` 合并行为回归
- **关联 AC**: AC3
- **类型**: 正向/边界
- **测试数据**: `testdata/python/20260307-refactor-pymerge-py311`
- **前置条件**:
  - 当前仓库根目录可执行 `python3`
  - 已存在 `./maze` 可执行入口
  - 使用 `cmd/maze-gen-coredump.py` 生成本目录 tarball
- **测试步骤**:
  1. 运行测试数据生成脚本，创建大量拥有 `refcnt=1` 子对象的实例。
  2. 使用 `cmd/maze-gen-coredump.py` 生成 tarball。
  3. 通过统一入口运行带 `--py-merge` 的分析。
  4. 检查父实例数量未发生翻倍，且嵌套子对象被稳定归并。
- **预期结果**:
  - 父对象 `MergeRoot` 可被识别。
  - `amount` 接近预设实例数量，不出现明显重复统计。
  - 总结果结构合法，执行过程中不出现 panic。
- **验证方式**: 自动化测试命令 + `validate.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260307-refactor-pymerge-py311 -- --py-merge
  ```

### TC-003: Python 3.11 managed dict / `__slots__` 兼容性回归
- **关联 AC**: AC4
- **类型**: 兼容性/边界
- **测试数据**: `testdata/python/20260307-refactor-managed-dict-py311`
- **前置条件**:
  - Python 3.11 环境
  - 当前仓库根目录可执行 `./maze`
  - 使用 `cmd/maze-gen-coredump.py` 生成本目录 tarball
- **测试步骤**:
  1. 运行测试脚本，创建普通类对象、`__slots__` 对象、闭包、bound method、`functools.partial`。
  2. 使用 `cmd/maze-gen-coredump.py` 生成 tarball。
  3. 执行统一验收命令。
  4. 检查结果中是否能稳定识别 managed dict 类实例与 `__slots__` 类实例。
- **预期结果**:
  - 普通类与 `__slots__` 类对象均被识别。
  - 结果中能看到闭包/方法相关对象或其宿主实例。
  - 不出现 managed dict 回归导致的空结果、异常崩溃或严重漏数。
- **验证方式**: 自动化测试命令 + `validate.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260307-refactor-managed-dict-py311
  ```

### TC-004: 空容器与大容器边界回归
- **关联 AC**: AC5
- **类型**: 边界
- **测试数据**: 复用 `testdata/python/20260307-refactor-comprehensive-py311`
- **前置条件**: 同 TC-001
- **测试步骤**:
  1. 执行 TC-001 的验收命令。
  2. 检查结果中与空容器/大容器相关的宿主实例统计。
  3. 关注日志中是否存在异常、panic、明显越界或解析失败。
- **预期结果**:
  - 空容器与大容器场景不会导致 Maze 崩溃。
  - 结果依旧完整可解析。
- **验证方式**: 自动化测试命令 + 日志检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260307-refactor-comprehensive-py311
  ```

### TC-005: 非法/异常对象布局容错回归
- **关联 AC**: AC5
- **类型**: 反向/边界
- **测试数据**: `testdata/python/20260307-refactor-managed-dict-py311`
- **前置条件**: 同 TC-003
- **测试步骤**:
  1. 运行带复杂对象混合布局的测试数据。
  2. 执行统一验收命令。
  3. 检查 `maze.log` 与 `maze.py.log` 中是否出现新的 panic、field not found、结构损坏等严重异常。
- **预期结果**:
  - 分析过程完成。
  - 即使部分对象难以分类，也不会导致整体结果崩溃。
- **验证方式**: 自动化测试命令 + 日志检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260307-refactor-managed-dict-py311
  ```

## 额外覆盖说明

- **准确性验证**:
  - 通过综合对象图和 `py-merge` 场景，对已知实例数量进行阈值校验，验证重构前后统计行为不回归。
- **边界条件**:
  - 覆盖空容器、大容器、混合容器、弱引用、闭包、`__slots__`。
- **兼容性**:
  - 本轮新增用例重点覆盖 Python 3.11 场景；后续执行验收时可在 Python 2.7 / 3.x 环境分别运行同类测试以完成兼容性清单。
- **性能**:
  - 当前设计未单独创建 >1GB 压测数据；执行验收阶段如需性能签核，应新增大对象测试数据并使用同一 `run_test.py` 框架验收。
