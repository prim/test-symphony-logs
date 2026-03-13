# Test Cases

## 关联计划
Feature: prim/maze#136 prepare 重复结果字段收敛到 `common.S` / `common.Cpp`

## Test File(s)
- `qa/issue136/prepare_single_source_fields_test.go`：针对字段收敛重构的静态结构约束测试，覆盖结构定义、同步逻辑、残留引用与导出链路。

## Cases
1. `TestContextStructRemovesConvergedResultFields`：验证 `prepare/context.go` 中各 manager 结构体不再定义本轮计划要求收敛到 `S` / `Cpp` 的重复结果字段。
2. `TestFillCommonVarsNoLongerCopiesConvergedFields`：验证 `prepare/fill.go` 不再承担这些字段的同步搬运，只保留非等价字段转换逻辑。
3. `TestPreparePackageHasNoResidualReferencesToRemovedContextResultFields`：验证全仓库不再残留对已删除 `Context` 结果字段的读取与写入引用。
4. `TestCppSnapshotExportUsesCommonCppInsteadOfGdbMirror`：验证 `cpp.json` 快照导出直接读取 `common.Cpp`，不再回退到 `GDB.ToJson*` 镜像结果。

## Test Case 列表

### TC-001: Context 重复结果字段删除约束
- **关联 AC**: AC1, AC2
- **类型**: 正向
- **测试数据**: 无需 `testdata/`，静态解析 `prepare/context.go`
- **前置条件**: 仓库可被 Go 测试框架读取
- **测试步骤**:
  1. 解析 `prepare/context.go` 中的结构体定义。
  2. 检查 `GDBManager`、`SymbolsManager`、`CppAnalyzer`、`PythonLangManager`、`TcmallocManager`、`JemallocManager`、`MimallocManager`、`NodejsLangManager`、`AsiocoreManager`、`MosdManager`、`TxosManager`。
  3. 断言计划中列出的重复结果字段均已删除。
- **预期结果**: 所有候选字段仅存在于 `common.S` / `common.Cpp`，`prepare.Context` 侧对应结构体不再定义镜像字段。
- **验证方式**: 自动化 Go 测试
- **验收命令**:
  ```bash
  go test ./qa/issue136 -run TestContextStructRemovesConvergedResultFields
  ```

### TC-002: FillCommonVars 去同步化约束
- **关联 AC**: AC3, AC7
- **类型**: 正向
- **测试数据**: 无需 `testdata/`，静态读取 `prepare/fill.go`
- **前置条件**: `prepare/fill.go` 存在
- **测试步骤**:
  1. 读取 `prepare/fill.go`。
  2. 搜索计划中列出的 `ctx.* -> S/Cpp` 同步赋值语句。
  3. 断言这些赋值标记全部消失。
- **预期结果**: `FillCommonVars()` 不再同步完全等价字段，避免 map/slice 浅拷贝同步语义。
- **验证方式**: 自动化 Go 测试
- **验收命令**:
  ```bash
  go test ./qa/issue136 -run TestFillCommonVarsNoLongerCopiesConvergedFields
  ```

### TC-003: 已删除 Context 字段无残留引用
- **关联 AC**: AC4, AC6
- **类型**: 正向
- **测试数据**: 无需 `testdata/`，静态扫描全仓库 `.go` 文件
- **前置条件**: 仓库源码完整可读
- **测试步骤**:
  1. 遍历仓库内 Go 源文件（排除 `testdata/`、`.git/`、`tmp/` 与当前测试目录）。
  2. 搜索所有本轮删除的 `ctx.*` 结果字段引用。
  3. 断言仓库中不存在残留使用点。
- **预期结果**: 已收敛字段只通过 `S` / `Cpp` 读取，不再从 `Context` 回读。
- **验证方式**: 自动化 Go 测试
- **验收命令**:
  ```bash
  go test ./qa/issue136 -run TestPreparePackageHasNoResidualReferencesToRemovedContextResultFields
  ```

### TC-004: C++ 快照导出单一来源约束
- **关联 AC**: AC5, AC6
- **类型**: 正向
- **测试数据**: 无需 `testdata/`，静态读取 `prepare/prepare.go`
- **前置条件**: `prepare/prepare.go` 存在
- **测试步骤**:
  1. 读取 `prepare/prepare.go` 中的 `buildLegacyCppSnapshot()` 实现。
  2. 断言实现包含 `structToMap(Cpp)`。
  3. 断言实现不再依赖 `currentPrepareContext.GDB.ToJson*` 镜像字段组装 `cpp.json`。
- **预期结果**: `cpp.json` 导出统一由 `common.Cpp` 提供，字段名保持兼容且不再依赖额外镜像状态。
- **验证方式**: 自动化 Go 测试
- **验收命令**:
  ```bash
  go test ./qa/issue136 -run TestCppSnapshotExportUsesCommonCppInsteadOfGdbMirror
  ```

## Notes for Developer
- 本组测试刻意采用静态结构约束方式，适合在重构早期先卡住“字段删除、入口切换、残留引用、导出来源”四类高风险回归点。
- 当前测试预期会失败，这是 TDD 设计的一部分；待功能实现完成后，应全部转绿。
- 本次未新增 `testdata/` 动态用例，因为需求核心是数据模型收敛与代码接线切换，优先使用静态测试锁定实现边界更直接。
