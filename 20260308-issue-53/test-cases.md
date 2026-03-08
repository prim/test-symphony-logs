# Test Cases

## 关联计划
Feature: prim/maze#53 — common result item 和 type id 重构，直接切换到新设计

## Test Case 列表

### TC-001: TypeID 位域编码与解析
- **关联 AC**: AC1, AC2
- **类型**: 正向
- **测试数据**: 新增 Go 单元测试 `common/results/typeid_test.go`
- **前置条件**: 代码库已实现 `common/results.TypeID` 与 `MakeTypeID`
- **测试步骤**:
  1. 构造 Python 模块 TypeID。
  2. 分别校验 `Module()`、`Subtype()`、`Sequence()` 返回值。
  3. 校验 String 输出包含 module/subtype/seq 信息。
- **预期结果**: 编码结果可被完整无损解析，字符串输出可用于调试。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run 'TestTypeID'
  ```

### TC-002: TypeRegistry 注册与重复注册保护
- **关联 AC**: AC3
- **类型**: 正向/反向
- **测试数据**: 新增 Go 单元测试 `common/results/registry_test.go`
- **前置条件**: 代码库已实现 `TypeRegistry`
- **测试步骤**:
  1. 注册一个新类型。
  2. 查询该类型并校验名称与分类。
  3. 重复注册相同 TypeID。
  4. 校验默认 formatter 与自定义 formatter 行为。
- **预期结果**: 首次注册成功，重复注册返回错误，名称格式化符合预期。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run 'TestTypeRegistry'
  ```

### TC-003: SimpleStrategy 统计聚合
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: 新增 Go 单元测试 `common/results/collector_test.go`
- **前置条件**: 已实现 `Collector`、`SimpleStrategy`
- **测试步骤**:
  1. 初始化 collector 并使用 `SimpleStrategy`。
  2. 连续添加同一类型的多个对象。
  3. Finalize 并读取统计结果。
- **预期结果**: count / total / avg 正确，且无 top-N 依赖。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run 'TestCollectorSimpleStrategy'
  ```

### TC-004: TopNStrategy 保留最大 N 个对象
- **关联 AC**: AC5, AC9
- **类型**: 边界
- **测试数据**: 新增 Go 单元测试 `common/results/collector_test.go`
- **前置条件**: 已实现 `TopNStrategy`
- **测试步骤**:
  1. 初始化 `TopNStrategy{N: 3}`。
  2. 添加 5 个不同大小对象。
  3. Finalize 后检查 `TopObjects` 长度与顺序。
- **预期结果**: 仅保留最大的 3 个对象，按 size 降序排列，不依赖全局 `CurObjSize`。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run 'TestCollectorTopNStrategy'
  ```

### TC-005: Collector.Merge 合并局部结果
- **关联 AC**: AC6
- **类型**: 正向
- **测试数据**: 新增 Go 单元测试 `common/results/collector_test.go`
- **前置条件**: 已实现 `Collector.Merge`
- **测试步骤**:
  1. 构造两个局部结果集。
  2. 合并到全局 collector。
  3. 校验 count / total 叠加正确。
- **预期结果**: 合并后统计值等于所有局部结果之和。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run 'TestCollectorMerge'
  ```

### TC-006: Reset 清理状态
- **关联 AC**: AC7
- **类型**: 边界
- **测试数据**: 新增 Go 单元测试 `common/results/collector_test.go`
- **前置条件**: 已实现 `Collector.Reset`
- **测试步骤**:
  1. 添加对象并确认结果存在。
  2. 调用 Reset。
  3. 校验结果集为空。
  4. 再次添加对象并确认可正常工作。
- **预期结果**: Reset 后状态被完全清空，且 collector 可复用。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run 'TestCollectorReset'
  ```

### TC-007: Python 内置类型注册与动态分类缓存
- **关联 AC**: AC8
- **类型**: 正向
- **测试数据**: 新增 Go 单元测试 `python/results_classifier_test.go`
- **前置条件**: Python 模块接入新 `results` 注册表与分类器
- **测试步骤**:
  1. 初始化默认注册表。
  2. 注册 Python 内置类型。
  3. 连续用相同特征做两次动态分类。
  4. 校验返回 TypeID 一致，且注册表中可查询到名称。
- **预期结果**: 内置类型已注册；相同特征的动态分类命中缓存，不重复生成 TypeID。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./python -run 'TestResultsClassifier'
  ```

### TC-008: JSON 输出回归不出现旧 type_id 兼容残留
- **关联 AC**: AC9
- **类型**: 反向
- **测试数据**: 新增 Go 单元测试 `common/results/export_test.go`
- **前置条件**: 新结果导出路径已接入新设计
- **测试步骤**:
  1. 生成新 results 输出 JSON。
  2. 检查输出字段集合。
  3. 确认未暴露旧设计残留字段或旧聚合结构。
- **预期结果**: 输出契约基于新设计，不再依赖旧 `type_id` / 旧 `common.ResultItem` 结构残留。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run 'TestResultsJSONContract'
  ```

### TC-009: Python 复杂类型回归
- **关联 AC**: AC10
- **类型**: 兼容性
- **测试数据**: `testdata/python/20260129-complex-types-311`
- **前置条件**: 具备对应 tarball 与验证脚本
- **测试步骤**:
  1. 执行 Python 复杂类型测试。
  2. 检查 maze 结果与 validate.py 输出。
  3. 检查 `maze.log`、`maze.py.log` 无异常。
- **预期结果**: 复杂 Python 类型统计正确，无明显类型归类回退。
- **验证方式**: `run_test.py` 自动化回归
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-010: Python 对象合并回归
- **关联 AC**: AC10
- **类型**: 准确性
- **测试数据**: `testdata/python/20260201-py-merge`
- **前置条件**: 具备对应 tarball 与验证脚本
- **测试步骤**:
  1. 执行 Python merge 回归。
  2. 检查对象数量与大小统计。
  3. 对比重构前既有行为是否退化。
- **预期结果**: 新策略替换旧设计后，核心 merge 统计仍满足测试预期。
- **验证方式**: `run_test.py` 自动化回归
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260201-py-merge
  ```

### TC-011: Node.js 综合回归
- **关联 AC**: AC10
- **类型**: 兼容性
- **测试数据**: `testdata/nodejs/20260211-comprehensive`
- **前置条件**: 具备对应 tarball 与验证脚本
- **测试步骤**:
  1. 执行 Node.js 综合测试。
  2. 检查对象类型名称与数量统计。
  3. 检查日志无异常。
- **预期结果**: Node.js 结果输出保持稳定，无 type id 重构引入的模块错乱。
- **验证方式**: `run_test.py` 自动化回归
- **验收命令**:
  ```bash
  python3 testdata/run_test.py nodejs/20260211-comprehensive
  ```

### TC-012: Node.js 与 heapsnapshot 准确性对比回归
- **关联 AC**: AC10
- **类型**: 准确性
- **测试数据**: `testdata/nodejs/20260225-maze-vs-heapsnapshot`
- **前置条件**: 具备对应 tarball 与验证脚本
- **测试步骤**:
  1. 执行 Maze vs heapsnapshot 回归。
  2. 读取统计结果并检查误差范围。
- **预期结果**: 重构后准确性不低于既有基线。
- **验证方式**: `run_test.py` 自动化回归
- **验收命令**:
  ```bash
  python3 testdata/run_test.py nodejs/20260225-maze-vs-heapsnapshot
  ```

### TC-013: C++ malloc 基础回归
- **关联 AC**: AC10
- **类型**: 正向
- **测试数据**: `testdata/cpp/20260201-basic-malloc`
- **前置条件**: 具备对应 tarball 与验证脚本
- **测试步骤**:
  1. 执行 C++ 基础 malloc 测试。
  2. 检查类型统计与对象个数。
- **预期结果**: C++ 对象分类和结果汇总未因重构而损坏。
- **验证方式**: `run_test.py` 自动化回归
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260201-basic-malloc
  ```

## 说明

- 本轮设计同时覆盖新架构的 **单元测试** 与 issue 明确要求的 **多 testdata 回归**。
- 当前仓库尚未实现 `common/results` 新包，因此新增 Go 单测预期会在当前阶段失败，这是符合 TDD 预期的。
- 验收阶段必须额外检查 `maze.log`、`maze.py.log`，确认无异常日志。
