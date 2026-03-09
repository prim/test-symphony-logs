# Test Cases

## 关联计划
Feature: common result item 与 type id 全量重构（移除旧方案，直接切换到新设计）

## Test Case 列表

### TC-001: TypeID 位域编码与解析正确
- **关联 AC**: AC1 新设计使用统一 TypeID 位域，替代旧魔法数字区间
- **类型**: 正向
- **测试数据**: 新增 `common/results/typeid_test.go`
- **前置条件**: Go 单元测试环境可用
- **测试步骤**:
  1. 运行 `go test ./common/results -run TestMakeTypeIDEncodesModuleSubtypeAndSequence`
  2. 校验 module/subtype/sequence 解码结果
- **预期结果**: TypeID 能正确编码并解码 module、subtype、sequence
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run TestMakeTypeIDEncodesModuleSubtypeAndSequence
  ```

### TC-002: TypeID 模块判断方法可替代旧区间判断
- **关联 AC**: AC1 新设计提供 O(1) 模块判断，不再依赖 `CustomClassifyTypeBegin` 等旧常量
- **类型**: 正向/边界
- **测试数据**: 新增 `common/results/typeid_test.go`
- **前置条件**: Go 单元测试环境可用
- **测试步骤**:
  1. 运行 `go test ./common/results -run TestTypeIDModulePredicates`
  2. 分别校验 Python/TXOS/Vtable/Nodejs 类型判断
- **预期结果**: 仅目标模块返回 true，其他模块返回 false
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run TestTypeIDModulePredicates
  ```

### TC-003: Collector + TopNStrategy 能保留最大对象并输出统计
- **关联 AC**: AC2 common result item 改为新 Collector/Strategy 设计，替代旧 `ResultItems`/`EnablePyMerge`/`CurObjSize`
- **类型**: 正向
- **测试数据**: 新增 `common/results/collector_test.go`
- **前置条件**: Go 单元测试环境可用
- **测试步骤**:
  1. 运行 `go test ./common/results -run TestCollectorTopNStrategyTracksLargestObjects`
  2. 校验 count/total/avg 与 TopN 排序结果
- **预期结果**: 统计值正确，TopObjects 仅保留最大 2 个对象且按降序排列
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run TestCollectorTopNStrategyTracksLargestObjects
  ```

### TC-004: 并行局部结果可通过 Merge 合并
- **关联 AC**: AC2 并行分析使用局部 `map[TypeID]*ResultItem` + `Merge()`
- **类型**: 正向
- **测试数据**: 新增 `common/results/collector_test.go`
- **前置条件**: Go 单元测试环境可用
- **测试步骤**:
  1. 运行 `go test ./common/results -run TestCollectorMergeCombinesParallelLocalResults`
  2. 校验多个局部 map 合并后的统计结果
- **预期结果**: 合并后的 count/total/avg 正确
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run TestCollectorMergeCombinesParallelLocalResults
  ```

### TC-005: 未注册类型可输出结构化未知类型名
- **关联 AC**: AC3 类型格式化统一走 registry，不再依赖旧 `FormatXxxTypeName`
- **类型**: 反向/边界
- **测试数据**: 新增 `common/results/collector_test.go`
- **前置条件**: Go 单元测试环境可用
- **测试步骤**:
  1. 运行 `go test ./common/results -run TestRegistryFormatsUnknownTypeIDsByDecodedFields`
  2. 校验未注册类型的兜底格式化输出
- **预期结果**: 输出包含 module/subtype 的结构化 unknown 字符串
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run TestRegistryFormatsUnknownTypeIDsByDecodedFields
  ```

### TC-006: Result String 输出可读的调试信息
- **关联 AC**: AC1 TypeID 新设计需具备可读性和可调试性
- **类型**: 正向
- **测试数据**: 新增 `common/results/typeid_test.go`
- **前置条件**: Go 单元测试环境可用
- **测试步骤**:
  1. 运行 `go test ./common/results -run TestTypeIDStringIncludesDecodedFields`
  2. 校验字符串中包含 module/subtype/seq
- **预期结果**: String 输出稳定且包含完整解码字段
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  go test ./common/results -run TestTypeIDStringIncludesDecodedFields
  ```

### TC-007: 回归验证现有 testdata 不因重构破坏输出
- **关联 AC**: AC4 QA 需要多执行几个 testdata 回归
- **类型**: 兼容性/回归
- **测试数据**: 现有 `testdata/python/20260129-complex-types-311`、`testdata/nodejs/20260225-maze-vs-heapsnapshot`、`testdata/cpp/20260201-basic-malloc`
- **前置条件**: testdata 子模块已初始化，具备对应运行环境
- **测试步骤**:
  1. 运行 Python 回归用例
  2. 运行 Node.js 准确性对比用例
  3. 运行 C++ malloc 基础用例
- **预期结果**: 重构后核心回归用例保持通过，输出 JSON 中 `items` 统计仍正确
- **验证方式**: `run_test.py` 自动化回归
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  python3 testdata/run_test.py nodejs/20260225-maze-vs-heapsnapshot
  python3 testdata/run_test.py cpp/20260201-basic-malloc
  ```

## Notes for Developer
- 这些测试按 TDD 方式设计，当前阶段预期失败，因为 `common/results` 新 API 还未落地。
- Issue 明确要求“不再保留现在的设计方案”，因此测试直接面向新包与新 API，故意不为旧 `common.ResultItem` / `CustomClassifyTypeBegin` / `EnablePyMerge` 提供兼容断言。
- 回归测试需要在功能实现后再完整执行；当前工作树中 `testdata/` 子模块未展开，已先在报告中固定验收命令。
