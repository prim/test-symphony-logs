# Test Cases

## 关联计划
Feature: `prim/maze#138` perf: optimize common mpm v2 hot paths

## Test Case 列表

### TC-001: Finalize 后禁止结构性修改
- **关联 AC**: AC2
- **类型**: 反向
- **测试数据**: `common/mpm/v2/hot_path_test.go::TestFinalizeDisallowsCollectAfterFinalize`
- **前置条件**: 可执行 Go 单元测试环境
- **测试步骤**:
  1. 创建并填充 `MemoryPieceManager`
  2. 调用 `Finalize()` 冻结结构
  3. 再次调用 `Collect()` 尝试写入新 piece
- **预期结果**: 结构性写入被拒绝（例如 panic），已 finalize 的查询态不被破坏
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/mpm/v2 -run TestFinalizeDisallowsCollectAfterFinalize -count=1
  ```

### TC-002: Finalize 后查询路径不依赖 manager 锁
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: `common/mpm/v2/hot_path_test.go::TestLookupDoesNotBlockOnManagerMutexAfterFinalize`
- **前置条件**: 可执行 Go 单元测试环境
- **测试步骤**:
  1. 创建并 `Finalize()` 一个只包含单页数据的 manager
  2. 在测试线程持有 `m.mu.Lock()`
  3. 并发调用 `SizeAndIsKnow()` 命中查询
  4. 观察查询是否在短超时内完成
- **预期结果**: 查询可在 manager 结构锁持有期间完成，说明 finalize 后热路径已去锁
- **验证方式**: 自动化单元测试 + 超时断言
- **验收命令**:
  ```bash
  go test ./common/mpm/v2 -run TestLookupDoesNotBlockOnManagerMutexAfterFinalize -count=1
  ```

### TC-003: 小页集合 IterPage 走串行 fast path
- **关联 AC**: AC3
- **类型**: 边界
- **测试数据**: `common/mpm/v2/hot_path_test.go::TestIterPageUsesSerialFastPathForSmallPageSets`
- **前置条件**: 可执行 Go 单元测试环境
- **测试步骤**:
  1. 创建仅 1 页的 finalized manager
  2. 启动 `IterPage()`，在回调中阻塞
  3. 统计执行期间新增 goroutine 数量
- **预期结果**: 小页场景不会 fan-out 出整组 worker，额外 goroutine 保持在极低水平
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/mpm/v2 -run TestIterPageUsesSerialFastPathForSmallPageSets -count=1
  ```

### TC-004: mostly-single-page Collect 分配基线
- **关联 AC**: AC5
- **类型**: 性能
- **测试数据**: `common/mpm/v2/hot_path_benchmark_test.go::BenchmarkCollectMostlySinglePage`
- **前置条件**: 可执行 Go benchmark 环境
- **测试步骤**:
  1. 构造 mostly-single-page 批量 piece 数据
  2. 重复创建 manager 并执行 `Collect()`
  3. 记录 `ns/op` 与 `allocs/op`
- **预期结果**: 后续实现应能在该基准上体现更低中间分配
- **验证方式**: benchmark 基线对比
- **验收命令**:
  ```bash
  go test ./common/mpm/v2 -run ^$ -bench BenchmarkCollectMostlySinglePage -benchmem -count=1
  ```

### TC-005: cross-page-heavy Collect 对照基线
- **关联 AC**: AC5
- **类型**: 性能
- **测试数据**: `common/mpm/v2/hot_path_benchmark_test.go::BenchmarkCollectCrossPageHeavy`
- **前置条件**: 可执行 Go benchmark 环境
- **测试步骤**:
  1. 构造大量跨页 piece 数据
  2. 重复执行 `Collect()`
  3. 对比对照组的时间与分配
- **预期结果**: 优化不会以牺牲跨页场景为代价
- **验证方式**: benchmark 基线对比
- **验收命令**:
  ```bash
  go test ./common/mpm/v2 -run ^$ -bench BenchmarkCollectCrossPageHeavy -benchmem -count=1
  ```

### TC-006: Finalize 后并行查询微基准
- **关联 AC**: AC1, AC4
- **类型**: 性能
- **测试数据**: `common/mpm/v2/hot_path_benchmark_test.go::BenchmarkLookupAfterFinalizeParallel`
- **前置条件**: 可执行 Go benchmark 环境
- **测试步骤**:
  1. 构造 finalized manager
  2. 用 `RunParallel` 并行调用 `SizeAndIsKnow()`
  3. 记录查询吞吐与分配
- **预期结果**: 后续实现应在 query-heavy 场景上显示更低常数开销
- **验证方式**: benchmark 基线对比
- **验收命令**:
  ```bash
  go test ./common/mpm/v2 -run ^$ -bench BenchmarkLookupAfterFinalizeParallel -benchmem -count=1
  ```

### TC-007: 单页 IterPage 调度基线
- **关联 AC**: AC3, AC4
- **类型**: 性能
- **测试数据**: `common/mpm/v2/hot_path_benchmark_test.go::BenchmarkIterPageSinglePage`
- **前置条件**: 可执行 Go benchmark 环境
- **测试步骤**:
  1. 构造只有 1 页的 finalized manager
  2. 反复执行 `IterPage()`
  3. 记录调度相关时间与分配
- **预期结果**: small-page fast path 实现后该基准应显著改善
- **验证方式**: benchmark 基线对比
- **验收命令**:
  ```bash
  go test ./common/mpm/v2 -run ^$ -bench BenchmarkIterPageSinglePage -benchmem -count=1
  ```

## 备注

- 当前新增的 3 个单元测试是 TDD 入口，预期在功能实现前失败。
- benchmark 用例不直接判定 PASS/FAIL，而是为开发与验收提供量化基线。
- 由于本 issue 聚焦 `common/mpm/v2` 热路径优化，本轮测试主要落在 Go 单元测试与 benchmark，而不是 `testdata/` coredump 用例。
