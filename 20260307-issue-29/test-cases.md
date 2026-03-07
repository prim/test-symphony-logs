# Test Cases

## 关联计划
Feature: 分析 common ResultItem 及其上层使用

## 测试策略说明

`dev-log/plan.md` 已明确：本 issue 是**文档分析任务**，交付物应为新的分析类 dev log，而不是功能实现、自动化测试资产或新的运行期行为。

因此本次 testcase 设计采用**文档验收型用例**，重点验证：

- 文档范围是否与 issue 一致
- 文档结论是否与源码实现一致
- 是否覆盖 `common.ResultItem` 的定义、聚合主链路、上游生产者、下游消费者
- 是否正确区分运行期聚合结构与输出阶段局部结构
- 是否说明 `EnablePyMerge`、`Http` 等分支对 `Objects` / `ObjSize` 的影响边界
- 是否保持“只交付文档、不新增实现/测试资产”的边界

本次**不新增 `test_*.py`、`testdata/`、`validate.py`**，因为这会违背本 issue 的交付范围，也不符合 `plan.md` 对 AC7 的要求。

## 源码核对范围

- **核心聚合结构**: `common/result_items.go`
- **排序与展示**: `common/webui.go`
- **文本 / JSON 输出适配**: `common/utils.go`
- **HTTP 消费链路**: `http.go`
- **Postman 消费链路**: `postman/postman-profile-main.go`
- **主要上游生产者**: `python/`、`nodejs/`、`txos/`、`mallocer/`

> 说明：本文件中的“验收命令”统一记为“无”，原因不是缺少测试设计，而是该 issue 的 AC 明确限定为**仅交付文档分析**。本次验收以**源码对照 + 文档审阅**完成，而非 `run_test.py` 自动化执行。

## Test Cases Designed

### Test File(s)
- 无新增可执行测试文件：本 issue 为文档分析任务，仅设计文档验收用例。

### Cases

1. **TC-001 文档范围与 issue 目标一致**：验证新 dev log 聚焦 `common.ResultItem` 定义、职责、数据流和上下游使用，不偏离到功能实现、修复方案或重构设计。
2. **TC-002 ResultItem 定义与核心字段说明完整**：验证文档准确说明 `common/result_items.go` 中 `ResultItem`、`ResultItems`、`ResultItemPool` 及字段 `Amount`、`Sizeof`、`Objects`、`ObjSize` 的职责。
3. **TC-003 聚合主链路描述正确**：验证文档正确串联 `NewResultItem`、`GetResultItem`、`AddToResultItems`、`AddToResultItems1`、`MergeResultItems`、`AddResult` 的关系，以及局部聚合到全局合并的数据流。
4. **TC-004 上游调用方覆盖充分**：验证文档覆盖主要生产者模块，如 `python/`、`nodejs/`、`txos/`、`mallocer/`，并描述常见调用模式：局部 map + 闭包 `GetResultItem` + 最终 `MergeResultItems`。
5. **TC-005 下游消费链路覆盖完整**：验证文档覆盖 `SortByValue`、`PrintSortedResults`、`PrintObjects`、`PrintTextModeResults` 以及 `http.go`、`postman/postman-profile-main.go` 的消费路径。
6. **TC-006 区分共享聚合结构与输出局部结构**：验证文档明确区分 `common/result_items.go` 中真正的 `common.ResultItem`，与 `common/utils.go` 的 `PrintTextModeResults` 内部局部输出结构。
7. **TC-007 排序与对象样本展示边界说明正确**：验证文档说明 `Objects` / `ObjSize` 不仅参与统计聚合，也会被 `PrintObjects` 等二级展示链路消费，并指出 `PrintSortedResults` 对 `ResultItems` 的裁剪边界。
8. **TC-008 EnablePyMerge / Http 分支影响说明正确**：验证文档说明 `EnablePyMerge` 与 `Http` 两个分支如何改变 `Objects`、`ObjSize`、TopN 保留与结果裁剪行为。
9. **TC-009 交付物边界正确**：验证本 issue 最终仅新增分析文档，不新增功能实现改动，也不新增自动化测试资产。

## 详细 Test Case 列表

### TC-001: 文档范围与 issue 目标一致
- **关联 AC**: AC1, AC7
- **类型**: 正向
- **测试数据**: `dev-log/plan.md`，开发者新增的 dev log
- **前置条件**:
  - `dev-log/plan.md` 已存在
  - 本 issue 已新增一份分析类 dev log
- **测试步骤**:
  1. 阅读 `dev-log/plan.md` 中的 issue、需求拆解、验收标准。
  2. 阅读开发者新增的 dev log。
  3. 核对文档是否聚焦 `common.ResultItem` 的定义位置、职责、数据流与上下游关系。
  4. 确认文档没有扩展到无关功能设计、实现方案、bug 修复或重构建议。
- **预期结果**: 新 dev log 与 issue 范围一致，仅交付分析文档。
- **验证方式**: 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-002: ResultItem 定义与核心字段说明完整
- **关联 AC**: AC1, AC2
- **类型**: 正向
- **测试数据**: `common/result_items.go`，开发者新增的 dev log
- **前置条件**:
  - 可以读取源码与 dev log
- **测试步骤**:
  1. 打开 `common/result_items.go`。
  2. 定位 `type ResultItem struct`、`ResultItems`、`ResultItemPool`。
  3. 检查 dev log 是否说明 `Amount`、`Sizeof`、`Objects`、`ObjSize` 的含义。
  4. 检查 dev log 是否说明 `ResultItems` 是全局聚合容器，`ResultItemPool` 用于对象复用。
- **预期结果**: 文档准确解释定义位置、字段语义和容器职责。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-003: 聚合主链路描述正确
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: `common/result_items.go`，开发者新增的 dev log
- **前置条件**:
  - 可以读取源码与 dev log
- **测试步骤**:
  1. 定位 `NewResultItem`、`GetResultItem`、`AddToResultItems`、`AddToResultItems1`、`MergeResultItems`、`AddResult`。
  2. 检查文档是否逐一说明这些函数的职责与关系。
  3. 检查文档是否说明主链路：局部 map 聚合 → 更新 `Amount` / `Sizeof` / `Objects` / `ObjSize` → `MergeResultItems` 合入全局 `ResultItems`。
  4. 检查文档是否说明互斥锁保护、对象池回收、`EnablePyMerge` / `Http` 分支行为。
- **预期结果**: 文档准确描述 ResultItem 从创建到全局合并的主数据流。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-004: 上游调用方覆盖主要调用模式
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: `python/`、`nodejs/`、`txos/`、`mallocer/` 中 `AddToResultItems` / `MergeResultItems` / `GetResultItem` 的调用点，开发者新增的 dev log
- **前置条件**:
  - 可以检索全仓库调用位置
- **测试步骤**:
  1. 搜索 `AddToResultItems(`、`AddToResultItems1(`、`MergeResultItems(`、`GetResultItem(`。
  2. 核对文档是否覆盖主要上游模块，而不是只分析 `common/` 内部。
  3. 检查文档是否概括典型模式：模块内创建 `map[uint64]*ResultItem`，通过闭包访问 `GetResultItem`，最终统一 `MergeResultItems`。
  4. 检查文档是否忠实反映真实分布，不夸大未发现的调用范围。
  5. 特别检查文档是否至少提及 `mallocer/` 侧也存在直接生产 `ResultItem` 数据的调用点。
- **预期结果**: 文档覆盖主要生产者与典型调用模式。
- **验证方式**: 代码检索 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-005: 下游消费与输出链路覆盖完整
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: `common/webui.go`、`common/utils.go`、`http.go`、`postman/postman-profile-main.go`，开发者新增的 dev log
- **前置条件**:
  - 可以读取输出相关源码与 dev log
- **测试步骤**:
  1. 阅读 `SortByValue`、`PrintSortedResults`、`PrintObjects`。
  2. 阅读 `PrintTextModeResults` 与 JSON 输出逻辑。
  3. 检查 `http.go`、`postman/postman-profile-main.go` 的调用路径。
  4. 检查文档是否说明聚合结果如何进入排序、展示、导出。
- **预期结果**: 文档清晰描述从聚合到展示/导出的主链路及复用关系。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-006: 区分共享聚合结构与文本/JSON 输出局部结构
- **关联 AC**: AC6
- **类型**: 边界
- **测试数据**: `common/result_items.go`、`common/utils.go`，开发者新增的 dev log
- **前置条件**:
  - 可以读取源码与 dev log
- **测试步骤**:
  1. 确认 `common/result_items.go` 中的 `common.ResultItem` 是共享聚合结构。
  2. 确认 `common/utils.go` 中 `PrintTextModeResults` 内定义了局部输出结构。
  3. 检查 dev log 是否明确区分这两类结构的生命周期和用途。
  4. 检查文档是否避免把输出结构误写成运行期聚合模型。
- **预期结果**: 文档对两类结构的职责区分清晰，无语义混淆。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-007: 排序与对象样本展示边界说明正确
- **关联 AC**: AC5
- **类型**: 边界
- **测试数据**: `common/webui.go`，开发者新增的 dev log
- **前置条件**:
  - 可以读取排序与对象展示相关源码
- **测试步骤**:
  1. 检查 `SortByValue` 的排序依据。
  2. 检查 `PrintSortedResults` 如何填充 `SortedResultMap`、截断 TopN、在非 HTTP 场景裁剪 `ResultItems`。
  3. 检查 `PrintObjects` 如何根据排序序号回查类型并消费 `Objects`。
  4. 检查文档是否覆盖这些边界。
- **预期结果**: 文档准确说明一级结果排序与二级对象展示之间的关系及边界。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-008: EnablePyMerge / Http 分支影响说明正确
- **关联 AC**: AC3, AC5
- **类型**: 边界
- **测试数据**: `common/result_items.go`、`common/webui.go`，开发者新增的 dev log
- **前置条件**:
  - 可以读取聚合与展示相关源码
- **测试步骤**:
  1. 阅读 `AddToResultItems` 中 `EnablePyMerge`、`Http`、默认分支对 `Objects` / `ObjSize` 的处理差异。
  2. 阅读 `MergeResultItems` 中 `EnablePyMerge` 分支对优先队列、`ObjSize` 收缩和 TopN 样本保留逻辑。
  3. 阅读 `PrintSortedResults` 中 `!Http` 时对 `ResultItems` 的裁剪逻辑。
  4. 检查 dev log 是否准确说明这些分支的边界与影响，而不是笼统描述“保存对象样本”。
- **预期结果**: 文档准确说明分支差异，包括对象样本采集、对象大小映射保留以及 HTTP / 非 HTTP 场景下的结果集合边界。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-009: 交付物边界正确，不引入额外实现或测试资产
- **关联 AC**: AC7
- **类型**: 反向
- **测试数据**: 本 issue 全部变更文件
- **前置条件**:
  - 已获取完整变更列表
- **测试步骤**:
  1. 检查变更文件列表。
  2. 确认没有修改 `.go` / `.py` 功能实现代码。
  3. 确认没有新增 `testdata/`、`validate.py`、`test_*.py` 等测试资产。
  4. 确认交付物主要是分析类 dev log。
- **预期结果**: 本 issue 保持文档分析边界，仅交付文档相关文件。
- **验证方式**: 变更列表检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

## Notes for Developer

- 重点不要只写 `common/result_items.go` 内部实现，要把上游生产者和下游消费链路串起来。
- 文档中应特别强调：`common.ResultItem` 是运行期聚合结构，而 `common/utils.go` 中的局部结构只是输出格式适配层。
- 建议用“定义 -> 聚合 -> 合并 -> 排序 -> 展示/导出”的顺序组织 dev log，便于验收。
- 本 issue 不应新增任何功能实现代码或自动化测试文件。

## 结论

- 本 issue **不应新增可执行测试代码**。
- 本 issue 的 testcase 以**文档覆盖性、分析准确性、调用链完整性、边界清晰性**为核心。
- 后续验收应以**源码对照 + 文档审阅**为主，而不是运行 `testdata/run_test.py`。
