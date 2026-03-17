# Test Cases

## 关联计划
Feature: prim/maze#160 vm: remove runtime switching from facade to eliminate acquireSnapshot hot path

## Test File(s)
- `qa/issue160/vm_facade_staticization_test.go`: 基于源码静态约束的 TDD 用例，覆盖 facade 热路径、运行时切换入口、兼容层语义和遗留测试迁移要求。

## Test Case 列表

### TC-001: facade 热路径移除 snapshot 与 atomic.Add
- **关联 AC**: AC1 `common/vm/facade.go` 不再在 `GetMemory*` 热路径上执行 `atomic.Add`
- **类型**: 正向 + 性能约束
- **测试数据**: 直接检查 `common/vm/facade.go`
- **前置条件**: 仓库包含 issue #160 的实现修改
- **测试步骤**:
  1. 读取 `common/vm/facade.go`
  2. 断言已删除 `implementationSnapshot`、`acquireSnapshot`、`releaseSnapshot`、`state.Add(...)`
  3. 断言 `dispatchGetMemory*` / `dispatchGetMemoryRange` 改为直接读取当前实现
- **预期结果**: facade 热路径不再依赖 snapshot 生命周期协议和原子加减
- **验证方式**: `go test ./qa/issue160 -run TestFacadeHotPathDropsSnapshotAtomicProtocol`

### TC-002: facade 不再暴露运行时切换和关闭入口
- **关联 AC**: AC3 相关 facade 契约测试更新并通过
- **类型**: 反向
- **测试数据**: 直接检查 `common/vm/facade.go`
- **前置条件**: 仓库包含 issue #160 的实现修改
- **测试步骤**:
  1. 读取 `common/vm/facade.go`
  2. 断言 `SwitchToLRU`、`SwitchToFull`、`SwitchToV3`、`CloseCurrent`、`switchImplementation` 已删除
- **预期结果**: facade API 不再支持运行时切换/关闭当前实现
- **验证方式**: `go test ./qa/issue160 -run TestFacadeRemovesRuntimeSwitchAndCloseEntrypoints`

### TC-003: SetVMType 语义收敛为仅初始化阶段生效
- **关联 AC**: AC3 相关 QA / facade 契约测试更新并通过
- **类型**: 正向 + 错误处理
- **测试数据**: 直接检查 `common/vm.go`
- **前置条件**: 仓库包含 issue #160 的实现修改
- **测试步骤**:
  1. 读取 `common/vm.go`
  2. 断言保留 `SetVMType`
  3. 断言其不再调用 `vm.SwitchTo*`
  4. 断言初始化路径仍通过 `selectPreferredVMType()` + `vm.Init()` 固定实现
- **预期结果**: `SetVMType` 不再表示运行时切换，仅服务初始化阶段选择
- **验证方式**: `go test ./qa/issue160 -run TestCompatLayerSetVMTypeStopsRuntimeSwitching`

### TC-004: 遗留 lifecycle / dispatch 契约测试完成迁移
- **关联 AC**: AC2 `go test ./common/vm/...` 通过；AC3 契约测试更新并通过
- **类型**: 回归
- **测试数据**: `common/vm/facade_lifecycle_test.go`、`common/vm/facade_dispatch_test.go`、`common/vm/v3_select_test.go`、`common/vm/init_v3_integration_test.go`、`common/vm_init_integration_test.go`
- **前置条件**: 仓库包含 issue #160 的实现修改
- **测试步骤**:
  1. 读取上述测试文件
  2. 断言不再引用 `SwitchTo*`、`CloseCurrent`、`boundImplementationSnap`、`acquireSnapshot`
- **预期结果**: 原有围绕动态切换/snapshot 的测试已被删除或替换为初始化期固定实现测试
- **验证方式**: `go test ./qa/issue160 -run TestLegacySwitchLifecycleTestsAreRemovedOrReplaced`

### TC-005: 需求与性能验收目标保持一致
- **关联 AC**: AC4 `pprof` 中 `acquireSnapshot` / `sync/atomic.(*Uint64).Add` 不再是全局热点；AC5 `refs=v0` / `refs=v1` 端到端耗时收敛
- **类型**: 文档/约束
- **测试数据**: `.symphony_prompt.md`
- **前置条件**: 需求文档已落盘
- **测试步骤**:
  1. 读取 `.symphony_prompt.md`
  2. 断言需求保留“仅初始化阶段生效”和 pprof 热点消失等验收点
- **预期结果**: 开发与后续 QA 可依据同一性能目标验收
- **验证方式**: `go test ./qa/issue160 -run TestFacadeDocsAndBenchExpectationsDescribeStaticInitialization`

## Cases
1. `TestFacadeHotPathDropsSnapshotAtomicProtocol`: 校验 facade 热路径删除 snapshot 原子协议并改为直接分发。
2. `TestFacadeRemovesRuntimeSwitchAndCloseEntrypoints`: 校验 facade 移除运行时切换与关闭入口。
3. `TestCompatLayerSetVMTypeStopsRuntimeSwitching`: 校验 `SetVMType()` 收敛为初始化阶段语义。
4. `TestLegacySwitchLifecycleTestsAreRemovedOrReplaced`: 校验旧 runtime-switch 测试已迁移。
5. `TestFacadeDocsAndBenchExpectationsDescribeStaticInitialization`: 校验需求与性能验收目标未丢失。

## Notes for Developer
- 这些测试为 TDD 约束，当前分支预期失败，因为代码仍保留 runtime switch 和 snapshot 生命周期协议。
- 通过标准应至少包括：`go test ./qa/issue160` 与 `go test ./common/vm/...`。
- 由于性能热点验收依赖 `pprof` 与端到端复测，本次仅先用静态测试锁定代码结构与语义方向。
