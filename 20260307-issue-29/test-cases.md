# Test Cases

## 关联计划
Feature: 分析 common ResultItem 及其上层使用

## 说明

`dev-log/plan.md` 已明确说明本 issue 是**文档分析任务**，约束如下：

- QA 不需要参与运行测试
- Dev 不需要编写功能实现代码
- 交付物是新的 dev log 分析文档

因此，本次测试设计采用**文档验收型 test case**。目标不是新增 `testdata/` 或可执行测试，而是为后续验收提供一套可重复执行的源码对照检查清单，用于验证开发者交付的分析文档是否完整、准确、边界清晰。

## Test Case 列表

### TC-001: 文档范围与 issue 目标一致
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: `dev-log/plan.md`，开发者提交的新 dev log
- **前置条件**:
  - `dev-log/plan.md` 已存在
  - 本 issue 已新增一份 dev log 文档
- **测试步骤**:
  1. 阅读 `dev-log/plan.md` 中的 issue 描述、需求拆解、验收标准。
  2. 阅读开发者新增的 dev log。
  3. 核对文档是否聚焦于：定位 `common.ResultItem`、分析职责与数据流、梳理上层使用关系。
  4. 确认文档没有偏离到无关功能设计、代码重构或 bug 修复方案。
- **预期结果**: 文档范围与 issue 一致，核心目标是分析 `common.ResultItem` 及其上层使用，而不是实现或修改功能。
- **验证方式**: 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-002: ResultItem 定义、字段与全局容器说明完整
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: `common/result_items.go`，开发者提交的新 dev log
- **前置条件**:
  - 可以读取源码与 dev log
- **测试步骤**:
  1. 打开 `common/result_items.go`。
  2. 定位 `type ResultItem struct`、`ResultItems`、`ResultItemPool`。
  3. 检查 dev log 是否说明 `Amount`、`Sizeof`、`Objects`、`ObjSize` 的含义和作用。
  4. 检查 dev log 是否说明 `ResultItems` 是全局结果集合，`ResultItemPool` 用于复用 `ResultItem` 实例。
- **预期结果**: 文档明确解释 `common.ResultItem` 的定义位置、核心字段，以及它与全局结果集合/对象池的关系。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-003: 构造、累积与合并链路说明正确
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: `common/result_items.go`，开发者提交的新 dev log
- **前置条件**:
  - 可以读取源码与 dev log
- **测试步骤**:
  1. 在 `common/result_items.go` 中定位 `NewResultItem`、`GetResultItem`、`AddToResultItems`、`AddToResultItems1`、`MergeResultItems`。
  2. 检查 dev log 是否逐一解释这些函数的职责。
  3. 检查文档是否说明主链路：局部 map 中获取/创建 `ResultItem` → 累积 `Amount`/`Sizeof`/`Objects` → 合并进全局 `ResultItems`。
  4. 检查文档是否说明 `MergeResultItems` 的互斥锁保护、对象列表合并，以及回收至 `ResultItemPool` 的过程。
  5. 检查文档是否提到 `EnablePyMerge` / `Http` 分支对 `Objects`、`ObjSize` 处理方式的影响。
- **预期结果**: 文档能够准确串起 `ResultItem` 的创建、局部聚合、全局合并和复用回收逻辑，无关键遗漏。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-004: 上游生产者覆盖主要调用模式
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: `python/`、`txos/`、`nodejs/`、`mallocer/` 中对 `GetResultItem` / `AddToResultItems` / `MergeResultItems` 的使用点，开发者提交的新 dev log
- **前置条件**:
  - 可以检索全仓库调用点
- **测试步骤**:
  1. 搜索 `GetResultItem(`、`AddToResultItems(`、`AddToResultItems1(`、`MergeResultItems(` 的调用位置。
  2. 检查 dev log 是否至少覆盖主要上游生产者，而非只分析 `common/` 包内部。
  3. 核对文档是否说明代表性模式：模块内先创建 `map[uint64]*ResultItem`，再用闭包包装 `GetResultItem`，最后统一 `MergeResultItems`。
  4. 若某模块并未直接使用这些接口，检查文档是否描述真实发现范围，而不是笼统声称“所有模块都使用”。
- **预期结果**: 文档覆盖主要生产者数据流，能够准确概括跨模块的典型使用方式与真实分布情况。
- **验证方式**: 代码检索 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-005: 下游消费与输出链路覆盖完整
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: `common/webui.go`、`common/utils.go`、`http.go`、`postman/postman-profile-main.go`，开发者提交的新 dev log
- **前置条件**:
  - 可以读取输出相关源码与 dev log
- **测试步骤**:
  1. 阅读 `common/webui.go` 中的 `SortByValue`、`PrintSortedResults`、`PrintObjects`。
  2. 阅读 `common/utils.go` 中的 `PrintTextModeResults` 及 JSON 输出逻辑。
  3. 检查 `http.go` 和 `postman/postman-profile-main.go` 对 `PrintSortedResults` / `PrintTextModeResults` 的调用。
  4. 检查 dev log 是否说明 `ResultItems` 如何从聚合结果进入排序、TopN 截断、Web UI / HTTP / Postman / 文本输出。
- **预期结果**: 文档能清晰描述 `ResultItems` 从内存统计结果到下游展示/导出的主链路，并指出不同消费端的复用关系。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-006: 区分共享 ResultItem 与文本/JSON 输出局部结构
- **关联 AC**: AC6
- **类型**: 边界
- **测试数据**: `common/result_items.go`、`common/utils.go`，开发者提交的新 dev log
- **前置条件**:
  - 可以读取源码与 dev log
- **测试步骤**:
  1. 确认 `common/result_items.go` 中存在共享的 `common.ResultItem`。
  2. 确认 `common/utils.go` 的 `PrintTextModeResults` 内定义了局部 `ResultItem`、`JsonResult` 输出结构。
  3. 检查 dev log 是否明确区分两类同名/近似结构：前者用于运行期聚合，后者仅用于文本/JSON 序列化。
  4. 检查文档是否避免将输出专用局部结构误写为全局统计模型。
- **预期结果**: 文档清晰区分运行期共享结构与输出期局部结构，避免名称相同带来的语义混淆。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-007: 文档准确描述排序与对象展示的消费边界
- **关联 AC**: AC5
- **类型**: 边界
- **测试数据**: `common/webui.go`、`txos/results.go`、`python/results.go`，开发者提交的新 dev log
- **前置条件**:
  - 可以读取排序和对象展示相关源码
- **测试步骤**:
  1. 检查 `SortByValue` 的排序依据，以及 `PrintSortedResults` 中对 `SortedResultMap`、TopN、`ResultItems` 的处理。
  2. 检查 `PrintObjects` 如何通过排序序号回查到 `ResultItems` 中的具体类型项。
  3. 检查文档是否指出 `Objects`/`ObjSize` 不只是统计字段，还会被二级对象展示链路消费。
  4. 检查文档是否说明非 HTTP 场景下 `PrintSortedResults` 可能裁剪 `ResultItems` 的行为边界。
- **预期结果**: 文档不仅描述一级结果排序，还说明对象样本列表如何被下游二级展示逻辑消费。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-008: 交付物边界正确，不引入额外实现或测试资产
- **关联 AC**: AC7
- **类型**: 反向
- **测试数据**: 本 issue 的全部变更文件
- **前置条件**:
  - 已获取完整变更列表
- **测试步骤**:
  1. 检查变更文件列表。
  2. 确认没有顺带修改 `.go` / `.py` 等功能实现代码来“优化” `ResultItem`。
  3. 确认没有新增 `testdata/`、`validate.py`、`test_*.py` 等与本 issue 不匹配的测试资产。
  4. 确认交付物以 dev log 文档为主。
- **预期结果**: 本 issue 保持文档分析任务边界，交付物聚焦文档，不引入额外实现或不必要测试代码。
- **验证方式**: 变更列表检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

## 结论

- 本 issue **不应新增可执行测试代码**。
- 本 issue 的 test case 以**文档覆盖性、分析准确性、调用链完整性**为核心。
- 后续执行验收时，应以**源码对照 + 文档审阅**为主，而不是运行 `testdata/run_test.py`。
