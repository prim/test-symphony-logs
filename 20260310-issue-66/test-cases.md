# Test Cases

## 关联计划
Feature: prim/maze#66 重构：彻底删除旧方案（ResultItems/CustomClassifyTypeBegin等），全面切换到新设计

## Test File(s)
- `common/legacy_scheme_removal_test.go`: 扫描非测试 Go 源码，阻止旧方案标识符和旧分类边界常量继续存在。

## Test Case 列表

### TC-001: 删除旧结果聚合方案标识符
- **关联 AC**: AC1, AC3, AC4
- **类型**: 反向
- **测试数据**: 仓库中的非测试 Go 源码
- **前置条件**: 在仓库根目录执行测试
- **测试步骤**:
  1. 运行 `go test ./common -run TestLegacyResultAggregationIdentifiersRemoved`
  2. 测试递归扫描仓库内非测试 Go 文件
  3. 测试断言旧结果聚合标识符已全部删除
- **预期结果**: 若仍存在 `ResultItems`、`AddToResultItems`、`MergeResultItems`、`SortedResultMap` 等旧标识符，测试失败并列出残留文件；全部移除后测试通过。
- **验证方式**: Go 自动化测试
- **验收命令**:
  ```bash
  go test ./common -run TestLegacyResultAggregationIdentifiersRemoved
  ```

### TC-002: 删除旧分类边界常量
- **关联 AC**: AC2, AC3, AC4
- **类型**: 反向
- **测试数据**: 仓库中的非测试 Go 源码
- **前置条件**: 在仓库根目录执行测试
- **测试步骤**:
  1. 运行 `go test ./common -run TestLegacyCustomClassifyBoundaryConstantsRemoved`
  2. 测试递归扫描仓库内非测试 Go 文件
  3. 测试断言 `CustomClassifyTypeBegin` 及派生常量已全部删除
- **预期结果**: 若仍存在 `CustomClassifyTypeBegin`、`CustomClassifyTypePythonBegin`、`CustomClassifyTypeLpcBegin`、`CustomClassifyTypeNodejsBegin` 等旧常量，测试失败并列出残留文件；全部移除后测试通过。
- **验证方式**: Go 自动化测试
- **验收命令**:
  ```bash
  go test ./common -run TestLegacyCustomClassifyBoundaryConstantsRemoved
  ```

## Notes for Developer

- 测试刻意扫描“非测试 Go 源码”，目的是防止仅通过兼容包装或转发函数保留旧方案。
- 若重构引入新的结果聚合/分类模型，这些测试无需感知新 API 的名称；它们只关心旧方案是否被彻底移除。
- 当前仓库仍大量使用旧标识符，因此这些测试在重构前失败是预期行为。
