# Test Cases

## 关联计划
Feature: 性能分析：VM v1 vs v2 性能对比与 v3 优化建议

## Test Case 列表

### TC-001: 性能对比文档记录完整
- **关联 AC**: AC1（记录 v1/v2 的随机访问性能对比数据）
- **类型**: 正向
- **测试数据**: 仓库内文档 `dev-log/2026-03-16-refs-vm-performance-comparison.md`
- **前置条件**: 仓库包含本工单的性能分析文档
- **测试步骤**:
  1. 检查性能分析文档是否存在。
  2. 校验文档中包含 GetMemory8、GetMemory4、顺序扫描三类场景。
  3. 校验文档中明确出现 v1-lru、v2-full、提升倍数等结论。
- **预期结果**: 文档存在且完整记录 issue 描述中的关键性能数据与结论。
- **验证方式**: 自动化静态检查 `qa/issue159/vm_v3_performance_test.go`
- **验收命令**:
  ```bash
  go test ./qa/issue159 -run TestVMPerformanceComparisonDocRecordsRequiredMetrics
  ```

### TC-002: V3 设计文档覆盖四项优化方向
- **关联 AC**: AC2（提出可执行的 v3 优化方向）
- **类型**: 正向
- **测试数据**: 仓库内文档 `dev-log/2026-03-15-vm-v3-design.md`
- **前置条件**: 已提交 v3 设计文档
- **测试步骤**:
  1. 检查设计文档是否存在。
  2. 校验文档是否覆盖无锁热路径、页表查找、批量 API、预加载优化。
  3. 校验文档是否说明 v3 相对 v2 的性能目标。
- **预期结果**: 设计文档完整覆盖四项优化方向，并定义量化目标。
- **验证方式**: 自动化静态检查 `qa/issue159/vm_v3_performance_test.go`
- **验收命令**:
  ```bash
  go test ./qa/issue159 -run TestVMV3DesignDocCoversOptimizationDirections
  ```

### TC-003: VM v3 实现骨架存在
- **关联 AC**: AC3（为 v3 优化方向提供独立实现承载目录）
- **类型**: 正向
- **测试数据**: `common/vm/v3/`
- **前置条件**: 开发者已开始实现 v3
- **测试步骤**:
  1. 检查 `common/vm/v3/` 目录是否存在。
  2. 校验 `vm.go`、`pagetable.go`、`nocopy.go`、`batch.go` 是否存在。
- **预期结果**: v3 目录与核心文件齐全，可承载后续优化实现。
- **验证方式**: 自动化静态检查 `qa/issue159/vm_v3_performance_test.go`
- **验收命令**:
  ```bash
  go test ./qa/issue159 -run TestVMV3ScaffoldExists
  ```

### TC-004: VM 类型与版本选择支持 v3
- **关联 AC**: AC4（v3 作为可选 VM 版本可被构建和选择）
- **类型**: 兼容性
- **测试数据**: `common/vm/types.go`, `common/vm/select_*.go`
- **前置条件**: 门面层支持多版本实现
- **测试步骤**:
  1. 校验 `VMType` 中新增 v3 枚举。
  2. 校验版本选择文件存在 `vm_v3` build tag 或等价入口。
  3. 校验 v3 被注册到 VM 门面层。
- **预期结果**: v3 可作为独立实现被识别、编译并切换。
- **验证方式**: 自动化静态检查 `qa/issue159/vm_v3_performance_test.go`
- **验收命令**:
  ```bash
  go test ./qa/issue159 -run TestVMSelectionSupportsV3BuildTagAndRegistration
  ```

### TC-005: 对比基准测试覆盖 v3 主链路
- **关联 AC**: AC5（性能结论可被基准测试复现）
- **类型**: 正向
- **测试数据**: `common/vm/bench_compare_test.go`
- **前置条件**: 对比基准测试文件已更新
- **测试步骤**:
  1. 检查对比基准是否包含 v1、v2、v3 三个版本。
  2. 校验至少覆盖 GetMemory8、GetMemory4、顺序扫描、随机访问、并发访问。
  3. 校验基准中包含 v3 目标或结果记录位置。
- **预期结果**: 基准测试可统一比较三代 VM 在关键访问模式下的性能。
- **验证方式**: 自动化静态检查 `qa/issue159/vm_v3_performance_test.go`
- **验收命令**:
  ```bash
  go test ./qa/issue159 -run TestCompareBenchmarksIncludeV3AcrossCriticalScenarios
  ```

### TC-006: v3 暴露批量读取 API
- **关联 AC**: AC6（支持批量 API 优化方向）
- **类型**: 边界
- **测试数据**: `common/vm/v3/*.go`
- **前置条件**: v3 已添加批量读取能力
- **测试步骤**:
  1. 解析 v3 目录导出函数。
  2. 校验存在 `GetMemoryBatch8`、`GetMemoryBatch4`、`MemoryIterator`。
- **预期结果**: v3 对外暴露明确的批量/迭代接口，支撑扫描场景优化。
- **验证方式**: 自动化静态检查 `qa/issue159/vm_v3_performance_test.go`
- **验收命令**:
  ```bash
  go test ./qa/issue159 -run TestVMV3ExportsBatchAPIs
  ```

## Test File(s)

- `qa/issue159/vm_v3_performance_test.go`: issue #159 的静态验收测试，覆盖文档、目录骨架、版本选择与基准覆盖要求。

## Notes for Developer

- 当前测试按 TDD 方式先定义目标，预期在功能和文档未补齐前失败。
- 测试刻意检查 `dev-log/2026-03-16-refs-vm-performance-comparison.md`，因为 issue 已将其列为参考文档。
- 为避免过度约束实现细节，测试重点放在能力存在性、构建入口与基准覆盖面，不限定内部算法写法。
