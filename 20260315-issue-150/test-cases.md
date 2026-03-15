# Test Cases

## 关联计划
Feature: VM V3 高性能版本

## Test Case 列表

### TC-001: V3 build tag 与目录结构可用性
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 使用 Go 单元测试 `common/vm/v3_select_tag_test.go`
- **前置条件**: 仓库包含 `common/vm/v3/` 目录与 `select_v3.go`
- **测试步骤**:
  1. 使用 `vm_v3` build tag 编译 VM 相关测试。
  2. 验证 `VMV3` 实现已注册。
  3. 调用 `SwitchToV3()` 并检查关键入口函数已绑定。
- **预期结果**: 编译成功，`VMV3` 可被选择并成为当前实现。
- **验证方式**: `go test -tags vm_v3 ./common/vm/...`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-002: 无锁加载并发首访无竞态
- **关联 AC**: AC2
- **类型**: 兼容性
- **测试数据**: 使用 Go 单元测试 `common/vm/v3/v3_contract_test.go:TestConcurrentLazyLoadRaceFree`
- **前置条件**: 支持 `go test -race`
- **测试步骤**:
  1. 构造延迟加载的 V3 测试 VM。
  2. 多 goroutine 同时读取同一 segment 的首访地址。
  3. 使用 race detector 执行测试。
- **预期结果**: 无 data race，且 segment 只加载一次。
- **验证方式**: `go test -race -tags vm_v3 ./common/vm/v3 -run TestConcurrentLazyLoadRaceFree`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-003: 批量读取与 iterator API 合约
- **关联 AC**: AC8
- **类型**: 正向
- **测试数据**: 使用 Go 单元测试 `common/vm/v3/v3_contract_test.go`
- **前置条件**: V3 暴露 `GetMemoryBatch8/4`、`MemoryIterator`
- **测试步骤**:
  1. 调用 `GetMemoryBatch8/4` 处理正常结果缓冲区与短缓冲区。
  2. 调用 `NewMemoryIterator()` 迭代跨 segment 连续数据。
  3. 校验非法地址回退与边界行为。
- **预期结果**: 返回值与输出数组长度符合约定，iterator 可跨 segment 正常推进。
- **验证方式**: `go test -tags vm_v3 ./common/vm/v3 -run 'TestBatchReadAPIContract|TestBatchReadHandlesShortResultsBuffer|TestMemoryIteratorTraversesAcrossSegments|TestPageTableLookupAndFallback|TestGetMemoryRangeHandlesUnalignedAndCrossSegment'`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-004: V3 benchmark 覆盖必需场景
- **关联 AC**: AC8
- **类型**: 边界
- **测试数据**: 使用 `common/vm/v3/v3_contract_test.go` 中 benchmark
- **前置条件**: V3 benchmark 文件可被 Go benchmark 发现
- **测试步骤**:
  1. 检查随机访问、顺序扫描、并发访问 benchmark 是否存在。
  2. 检查未对齐、跨 segment、cache miss、安全读取、mixed workload 是否存在。
  3. 检查 Python/V8 场景 benchmark 是否存在。
- **预期结果**: benchmark 名称完整，覆盖 AC 指定场景。
- **验证方式**: `go test -tags vm_v3 ./common/vm/v3 -run '^$' -bench 'Benchmark(GetMemory8|GetMemory4|SequentialScan|RandomAccess|ConcurrentAccess|HotspotAccess|UnalignedAccess|CrossSegment|CacheMiss|GetMemorySafe|MixedWorkload|ColdStart|PythonObjectTraverse|V8HeapScan)$'`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-005: V3 与 V2 对比基准存在且可执行
- **关联 AC**: AC9
- **类型**: 性能
- **测试数据**: 使用 `common/vm/bench_compare_v3_test.go`
- **前置条件**: `bench_compare_v3_test.go` 引入 V2 与 V3
- **测试步骤**:
  1. 运行 V2 与 V3 的对比 benchmark。
  2. 检查测试中存在 V3 优于 V2 的断言占位。
  3. 后续实现完成后补充性能阈值验证。
- **预期结果**: 对比 benchmark 文件存在，覆盖至少随机访问主链路。
- **验证方式**: `go test -tags vm_v3 ./common/vm -run TestV3OutperformsV2Placeholders -bench BenchmarkCompareV3GetMemory8`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-006: Python / Node.js 主链路回归未退化
- **关联 AC**: AC7
- **类型**: 回归
- **测试数据**: `testdata/python/20260129-complex-types-311`、`testdata/nodejs/20260225-maze-vs-heapsnapshot`
- **前置条件**: 支持通过 `vm_v3` 构建 maze
- **测试步骤**:
  1. 构建启用 V3 的 maze。
  2. 运行 Python 主链路测试。
  3. 运行 Node.js 准确性对比测试。
- **预期结果**: 两类语言回归均通过，无明显统计退化。
- **验证方式**: `./maze --build` 后执行 `testdata/run_test.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-007: 页表内存开销限制
- **关联 AC**: AC6
- **类型**: 性能
- **测试数据**: 使用 Go 单元测试扩展 `v3_contract_test.go`
- **前置条件**: 暴露页表容量或统计接口
- **测试步骤**:
  1. 构造多 segment VM。
  2. 读取 V3 页表与缓存结构统计。
  3. 对比 V2 基线结构占用。
- **预期结果**: 额外内存开销低于 20%。
- **验证方式**: `go test -tags vm_v3 ./common/vm/v3 -run TestPageTableMemoryOverhead`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-008: 性能目标门槛校验
- **关联 AC**: AC3, AC4, AC5
- **类型**: 性能
- **测试数据**: V3 benchmark + compare benchmark
- **前置条件**: 基准运行环境稳定
- **测试步骤**:
  1. 执行随机访问 benchmark 并记录 ns/op。
  2. 执行顺序扫描 benchmark 并记录 ns/op。
  3. 执行并发访问 benchmark 并记录 ns/op。
  4. 与 V2 基线对比提升比例。
- **预期结果**: 三项指标满足 issue 中阈值。
- **验证方式**: `go test -tags vm_v3 ./common/vm/v3 ./common/vm -run '^$' -bench 'Benchmark(RandomAccess|SequentialScan|ConcurrentAccess|CompareV3GetMemory8)' -benchmem`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

## Notes for Developer
- 当前测试采用 TDD 方式，预期在功能未实现时编译失败或测试失败。
- 建议先补齐 `VMV3` 类型、`SwitchToV3`、`select_v3.go` 与 `common/vm/v3` 目录中的构造函数，再完善性能断言。
- `bench_compare_v3_test.go` 中保留了显式失败占位，用于提醒开发者补充真实比较逻辑与阈值检查。
