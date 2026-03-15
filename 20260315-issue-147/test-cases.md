# Test Cases

## 关联计划
Feature: VM 门面化重构：拆分为 facade + 版本实现

## Test Case 列表

### TC-001: 门面目录骨架与版本目录存在
- **关联 AC**: AC1, AC2, AC3
- **类型**: 正向
- **测试数据**: 无需 testdata，静态代码结构检查
- **前置条件**: 代码已拉取到本地工作树
- **测试步骤**:
  1. 检查 `common/vm/` 是否存在。
  2. 检查 `facade.go`、`types.go`、`select_default.go`、`select_lru.go`、`select_full.go`、`select_conflict.go`、`shared/helpers.go` 是否存在。
  3. 检查 `common/vm/v1` 与 `common/vm/v2` 目录是否存在。
- **预期结果**: VM 门面、共享层、版本实现层目录齐备。
- **验证方式**: `qa/issue147/vm_facade_migration_test.go::TestVMFacadePackageScaffoldExists`
- **验收命令**:
  ```bash
  go test ./qa/issue147 -run TestVMFacadePackageScaffoldExists
  ```

### TC-002: 编译期 build tags 选择规则正确
- **关联 AC**: AC5
- **类型**: 正向/边界
- **测试数据**: 无需 testdata，静态选择文件检查
- **前置条件**: 代码已拉取到本地工作树
- **测试步骤**:
  1. 检查默认选择文件是否使用 `!vm_lru && !vm_full`。
  2. 检查 LRU/FULL 选择文件是否分别绑定 `bindV1()` / `bindV2()`。
  3. 检查冲突文件是否声明双 tag 冲突保护。
- **预期结果**: 默认构建绑定 FULL，实现可由 build tags 切换，双 tag 同开时有冲突保护。
- **验证方式**: `qa/issue147/vm_facade_migration_test.go::TestVMVersionSelectionFilesUseBuildTags`
- **验收命令**:
  ```bash
  go test ./qa/issue147 -run TestVMVersionSelectionFilesUseBuildTags
  ```

### TC-003: v1/v2 导出 API 与门面绑定保持一致
- **关联 AC**: AC1, AC2, AC3
- **类型**: 正向/兼容性
- **测试数据**: 无需 testdata，AST 签名检查
- **前置条件**: 代码已拉取到本地工作树
- **测试步骤**:
  1. 检查 `common/vm/facade.go` 是否暴露 `GetMemory*`、`GetMemoryRange`、`VMStat`、`Init`、`CurrentVMType`。
  2. 提取 `common/vm/v1` 导出函数签名。
  3. 提取 `common/vm/v2` 导出函数签名并逐一对比。
- **预期结果**: v1/v2 暴露相同全局函数集合，门面可以无差别绑定。
- **验证方式**: `qa/issue147/vm_facade_migration_test.go::TestVMFacadeExportsUniformGlobalFunctions`
- **验收命令**:
  ```bash
  go test ./qa/issue147 -run TestVMFacadeExportsUniformGlobalFunctions
  ```

### TC-004: common/vm.go 收敛为兼容转发层
- **关联 AC**: AC4
- **类型**: 正向/回归
- **测试数据**: 无需 testdata，源文件内容检查
- **前置条件**: 代码已拉取到本地工作树
- **测试步骤**:
  1. 检查 `common/vm.go` 是否导入 `maze/common/vm`。
  2. 检查是否仅保留兼容类型与转发入口。
  3. 检查旧的 ELF、共享库、静态内存、段查找实现是否已迁出。
- **预期结果**: `common/vm.go` 只保留兼容层，不再承载主实现细节。
- **验证方式**: `qa/issue147/vm_facade_migration_test.go::TestCommonVMBecomesThinCompatLayer`
- **验收命令**:
  ```bash
  go test ./qa/issue147 -run TestCommonVMBecomesThinCompatLayer
  ```

### TC-005: 业务层不直接依赖 VM 版本包
- **关联 AC**: AC1
- **类型**: 反向
- **测试数据**: 无需 testdata，源码 import 扫描
- **前置条件**: 代码已拉取到本地工作树
- **测试步骤**:
  1. 扫描仓库中的非测试 Go 源文件。
  2. 排除 `common/vm` 自身与 `qa/issue147`。
  3. 校验业务层未直接导入 `maze/common/vm/v1`、`v2`、`shared`。
- **预期结果**: 业务层只依赖稳定门面，不直接依赖具体 VM 版本实现。
- **验证方式**: `qa/issue147/vm_facade_migration_test.go::TestBusinessPackagesAvoidDirectVMVersionImports`
- **验收命令**:
  ```bash
  go test ./qa/issue147 -run TestBusinessPackagesAvoidDirectVMVersionImports
  ```

### TC-006: 运行时切换兼容入口前移到门面层
- **关联 AC**: AC5
- **类型**: 正向/回归
- **测试数据**: 无需 testdata，兼容入口检查
- **前置条件**: 代码已拉取到本地工作树
- **测试步骤**:
  1. 检查 `common/vm/facade.go` 是否提供 `SwitchToLRU` 与 `SwitchToFull`。
  2. 检查 `common/vm.go` 中的 `SetVMType` 是否转发到门面层。
  3. 检查兼容层是否仍直接调用 `LRU_initVirtualMemory` / `FULL_initVirtualMemory`。
- **预期结果**: 运行时切换能力仍保留，但切换逻辑集中到门面层。
- **验证方式**: `qa/issue147/vm_facade_migration_test.go::TestVMRuntimeSwitchCompatibilityMovesToFacade`
- **验收命令**:
  ```bash
  go test ./qa/issue147 -run TestVMRuntimeSwitchCompatibilityMovesToFacade
  ```

## Notes for Developer

- 这些测试是 TDD 约束，当前分支上预期失败，原因是 VM 门面化目录与兼容转发尚未落地。
- 测试风格对齐 `qa/issue134` 与 `qa/issue140`：优先用静态结构和 AST 校验限制迁移范围，避免在重构早期引入高成本动态依赖。
- 如果共享逻辑拆分文件名不是 `shared/helpers.go`，可在实现时同步调整测试，但需保持“共享层独立于兼容层”的结构约束不变。
