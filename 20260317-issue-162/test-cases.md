# Test Cases

## 关联计划
Feature: vm: make facade compile-time-only and remove runtime implementation state

## Test Case 列表

### TC-001: facade 删除运行时 implementation 状态
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: `qa/issue162/vm_facade_compile_time_only_test.go`
- **前置条件**: 仓库包含 `common/vm/facade.go`
- **测试步骤**:
  1. 读取 `common/vm/facade.go`
  2. 校验 facade 不再声明 `implementation` / `implementationBinding` 等运行时状态结构
  3. 校验 facade 不再包含 `boundImplementation`、`currentVMType`、`initialized` 等状态字段
- **预期结果**: `common/vm` 不再维护当前 implementation 的运行时绑定状态
- **验证方式**: 自动化静态测试
- **验收命令**:
  ```bash
  go test ./qa/issue162 -run TestFacadeRemovesRuntimeImplementationState
  ```

### TC-002: GetMemory 热路径去除 runtime dispatch
- **关联 AC**: AC2, AC4
- **类型**: 正向
- **测试数据**: `qa/issue162/vm_facade_compile_time_only_test.go`
- **前置条件**: 仓库包含 `common/vm/facade.go`
- **测试步骤**:
  1. 读取 `common/vm/facade.go`
  2. 校验 `GetMemory*` 不再绑定到 `dispatchGetMemory*`
  3. 校验 facade 中不存在 `currentImplementation()`、nil 检查等运行时分发痕迹
  4. 校验文件中存在编译期直接转发或函数直绑形态标记
- **预期结果**: `GetMemory*` 不再通过运行时 dispatch 取实现，而是直接转发到编译期选中的实现
- **验证方式**: 自动化静态测试
- **验收命令**:
  ```bash
  go test ./qa/issue162 -run TestGetMemoryFacadeStopsRuntimeDispatch
  ```

### TC-003: compat 层停止运行时选择并明确重新编译切换
- **关联 AC**: AC1, AC2
- **类型**: 正向/兼容性
- **测试数据**: `qa/issue162/vm_facade_compile_time_only_test.go`
- **前置条件**: 仓库包含 `common/vm.go`
- **测试步骤**:
  1. 读取 `common/vm.go`
  2. 校验 `SetVMType` 不再依赖 `vm.HasImplementation`、`vm.SelectImplementation`、`vm.CurrentVMType`
  3. 校验兼容层文案或注释明确 VM 切换需重新编译完成
- **预期结果**: compat 层不再承担运行时 VM 选择逻辑，并明确通过重新编译切换实现
- **验证方式**: 自动化静态测试
- **验收命令**:
  ```bash
  go test ./qa/issue162 -run TestCompatLayerStopsRuntimeSelectionAndDocumentsRebuildOnlySwitch
  ```

### TC-004: vm_dual 被删除或降级为非常规模式
- **关联 AC**: AC3
- **类型**: 正向/边界
- **测试数据**: `qa/issue162/vm_facade_compile_time_only_test.go`
- **前置条件**: 仓库存在或不存在 `common/vm/select_dual.go`
- **测试步骤**:
  1. 检查 `common/vm/select_dual.go` 是否存在
  2. 若不存在，则视为删除完成
  3. 若存在，则校验其不再注册常规实现，也不再默认绑定 VM，并明确标注 debug/非常规语义
- **预期结果**: `vm_dual` 被移除，或仅保留为调试/非常规模式
- **验证方式**: 自动化静态测试
- **验收命令**:
  ```bash
  go test ./qa/issue162 -run TestVMDualIsRemovedOrExplicitlyDebugOnly
  ```

### TC-005: 各 build tag 选择文件只保留单实现直绑
- **关联 AC**: AC1, AC2
- **类型**: 正向/边界
- **测试数据**: `qa/issue162/vm_facade_compile_time_only_test.go`
- **前置条件**: 仓库包含 `common/vm/select_*.go`
- **测试步骤**:
  1. 读取 `select_default.go`、`select_lru.go`、`select_full.go`、`select_v3.go`
  2. 校验选择文件不再通过 `registerImplementation` 和 `bindV*` 进行运行时组织
  3. 校验选择文件不再通过共享状态写入版本信息
- **预期结果**: 每个编译产物仅保留一个实现，选择文件只负责编译期绑定而非运行时注册
- **验证方式**: 自动化静态测试
- **验收命令**:
  ```bash
  go test ./qa/issue162 -run TestCompileTimeSelectionFilesDirectlyBindSingleImplementation
  ```

## 测试文件

- `qa/issue162/vm_facade_compile_time_only_test.go`: 面向 issue #162 的静态契约测试，覆盖 facade 去状态化、去 runtime dispatch、compat 层收敛、`vm_dual` 收敛和 build-tag 选择文件改造。

## Notes for Developer

- 本轮仅新增测试，不实现功能。
- 这些测试基于 issue 与开发计划中的验收标准设计，当前应失败，以驱动后续实现。
- 如后续采用“薄 wrapper”方案，建议在 facade 中保留稳定 API，但不要重新引入任何运行时 implementation state。
