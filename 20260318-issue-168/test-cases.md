# Test Cases

## 关联计划
Feature: prim/maze#168 Refactor local ResultItem aggregation to use AddResult consistently

## Test Case 列表

### TC-001: 串行聚合调用点统一为 AddResult
- **关联 AC**: AC1 串行调用点不再手写局部聚合壳
- **类型**: 正向
- **测试数据**: 代码静态检查，无需新增 testdata
- **前置条件**: 仓库包含本次重构涉及文件
- **测试步骤**:
  1. 检查 `python/number_block.go`、`python/mosd.go`、`python/mobile_server2.go`、`python/mobile_server2_main.go`、`txos/txos-aoi.go`、`cpp/openblas.go`
  2. 验证这些文件使用 `AddResult(func(function func(s uint64) *ResultItem) { ... })`
  3. 验证这些文件不再保留 `resultItems := make(map[uint64]*ResultItem)` / `GetResultItem(..., resultItems)` / `MergeResultItems(resultItems)` 三件套
- **预期结果**: 串行路径统一通过 `AddResult` 聚合，业务调用仍使用原有 `function`
- **验证方式**: 自动化 Go 测试
- **验收命令**:
  ```bash
  go test ./qa/issue168 -run TestSerialAggregationCallSitesUseAddResult
  ```

### TC-002: IterPage 路径按 page 作用域使用 AddResult
- **关联 AC**: AC2 IterPage 路径保持 page 粒度，不共享局部 map
- **类型**: 边界
- **测试数据**: 代码静态检查，无需新增 testdata
- **前置条件**: 涉及 `IterPage` 的文件已纳入重构范围
- **测试步骤**:
  1. 检查 `python/maze.go`、`txos/leak.go`、`mallocer/pieces.go`
  2. 验证 `AddResult` 出现在 `IterPage` 回调内部
  3. 验证 `IterPage` 回调开头不再直接创建 `resultItems` 本地 map
  4. 验证 `BatchSetResultObjTypeMap` 等副作用代码仍在原有 page 作用域内
- **预期结果**: 每个 page 独立聚合，避免把 `AddResult` map 暴露给整轮并发扫描
- **验证方式**: 自动化 Go 测试
- **验收命令**:
  ```bash
  go test ./qa/issue168 -run TestIterPageScopesWrapPerPageAddResultOnly
  ```

### TC-003: worker 并行路径按 worker 作用域使用 AddResult
- **关联 AC**: AC3 worker 并行路径不共享一个 AddResult map
- **类型**: 兼容性
- **测试数据**: 代码静态检查，无需新增 testdata
- **前置条件**: 并行 worker 代码可正常读取
- **测试步骤**:
  1. 检查 `cpp/cpp.go`、`mallocer/tcmalloc/tcmalloc.go`、`mallocer/ptmalloc/ptmalloc.go`、`mallocer/mimalloc/mimalloc.go`
  2. 验证 worker / 局部扫描函数内部使用 `AddResult`
  3. 验证不再手写局部 `resultItems` 聚合壳
- **预期结果**: 每个 worker 或局部扫描单元单独聚合，消除共享 map 风险
- **验证方式**: 自动化 Go 测试
- **验收命令**:
  ```bash
  go test ./qa/issue168 -run TestWorkerScopedAggregationUsesAddResultWithoutCrossWorkerMapSharing
  ```

### TC-004: 显式手工构造 ResultItem 的路径保持不变
- **关联 AC**: AC4 手工构造完整 ResultItem 的路径不强行替换
- **类型**: 反向
- **测试数据**: 代码静态检查，无需新增 testdata
- **前置条件**: `common/c.go` 存在 `AddMiscToResultItems`
- **测试步骤**:
  1. 检查 `common/c.go`
  2. 验证函数仍手工 `NewResultItem()` 并 `MergeResultItems(resultItems)`
  3. 验证该函数未被改写为 `AddResult`
- **预期结果**: 范围外场景保持原实现，避免误改语义
- **验证方式**: 自动化 Go 测试
- **验收命令**:
  ```bash
  go test ./qa/issue168 -run TestExplicitManualResultItemConstructionRemainsUnchanged
  ```

### TC-005: 需求文档明确约束并发边界
- **关联 AC**: AC5 需求与实现边界一致
- **类型**: 错误处理
- **测试数据**: `.symphony_prompt.md` 与 `dev-flow/plan.md`
- **前置条件**: 已生成 `dev-flow/plan.md`
- **测试步骤**:
  1. 检查 issue 提示文档与 `dev-flow/plan.md`
  2. 验证都明确约束“不能跨 goroutine 共享一个 AddResult 局部 map”
  3. 验证都要求 `IterPage` / worker 路径按局部作用域使用 `AddResult`
- **预期结果**: 测试基线与开发实现边界一致，防止错误重构方向
- **验证方式**: 自动化 Go 测试
- **验收命令**:
  ```bash
  go test ./qa/issue168 -run TestIssuePromptAndPlanCaptureConcurrencyConstraint
  ```

## Notes for Developer

- 本组测试采用静态结构断言，目的是约束“重构范围”和“并发边界”，防止只做字符串替换却破坏 page / worker 粒度。
- 当前仓库仍保留多处手写聚合壳，这些测试在功能实现前失败属于预期行为。
- `common/c.go` 这类手工构造完整 `ResultItem` 的路径被明确视为保留项，避免误改。
