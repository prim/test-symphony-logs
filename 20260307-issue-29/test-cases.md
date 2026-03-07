# Test Cases

## 关联计划
Feature: 分析 common ResultItem 及其上层使用

## 说明

本 issue 在 `dev-log/plan.md` 中已明确标注为**文档分析任务**：

- QA 不需要参与运行验收测试
- Dev 不需要编写功能代码
- 交付物是新的 dev log 分析文档

因此，本次 test case 采用**文档验收型用例**，用于检查开发者提交的分析文档是否完整覆盖需求；**不新增 `testdata/`、`validate.py`、`run_test.py` 用例，也不要求新增可执行自动化测试代码**。

## Test Case 列表

### TC-001: 文档范围符合 issue 目标
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 开发者提交的新 dev log 文档
- **前置条件**:
  - `dev-log/plan.md` 已存在
  - 已有新的 dev log 可供检查
- **测试步骤**:
  1. 阅读 `dev-log/plan.md` 中的 issue 描述与验收关注点。
  2. 阅读开发者新增的 dev log。
  3. 核对文档目标是否聚焦于：定位 `common.ResultItem`、分析其职责与数据流、梳理上层使用关系。
  4. 确认文档没有偏离到无关功能开发或 bug 修复方案。
- **预期结果**: 新 dev log 的范围与 issue 一致，仅分析 `common.ResultItem` 及其上层使用，并作为文档交付物存在。
- **验证方式**: 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-002: ResultItem 定义与核心字段覆盖完整
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: `common/result_items.go`，开发者提交的新 dev log
- **前置条件**:
  - 可以读取源码与 dev log
- **测试步骤**:
  1. 打开 `common/result_items.go`。
  2. 定位 `type ResultItem struct`。
  3. 检查 dev log 是否说明 `Amount`、`Sizeof`、`Objects`、`ObjSize` 的含义与用途。
  4. 检查 dev log 是否解释 `ResultItems` 全局结果集合与 `ResultItemPool` 的角色。
- **预期结果**: dev log 明确说明 `common.ResultItem` 的定义位置、核心字段和它在结果聚合中的职责。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-003: 构造与聚合辅助函数关系说明正确
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: `common/result_items.go`，开发者提交的新 dev log
- **前置条件**:
  - 可以读取源码与 dev log
- **测试步骤**:
  1. 在 `common/result_items.go` 中定位 `NewResultItem`、`GetResultItem`、`AddToResultItems`、`AddToResultItems1`、`MergeResultItems`。
  2. 检查 dev log 是否逐一说明这些函数的职责。
  3. 检查 dev log 是否描述它们之间的调用关系：创建/获取局部项 → 累积 amount/sizeof/objects → 合并到全局 `ResultItems`。
  4. 检查 dev log 是否提到 `MergeResultItems` 的并发保护和对象池回收逻辑。
- **预期结果**: dev log 对辅助函数之间的关系描述准确，没有遗漏核心聚合链路。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-004: 上游生产者数据流覆盖主要模块
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: `python/`、`nodejs/`、`mallocer/`、`txos/` 中对 `ResultItem` 的调用点，开发者提交的新 dev log
- **前置条件**:
  - 可以搜索源码中的 `GetResultItem`、`AddToResultItems`、`MergeResultItems`
- **测试步骤**:
  1. 检索主要模块中对 `ResultItem` 聚合接口的使用。
  2. 检查 dev log 是否覆盖代表性上游调用者，而非只分析 `common/` 自身。
  3. 确认 dev log 是否说明常见模式：模块内部先构造 `map[uint64]*ResultItem`，通过闭包形式的 `GetResultItem` 聚合，再调用 `MergeResultItems` 合并。
  4. 如某模块无实际调用，检查文档是否明确说明真实发现范围，而不是笼统声称“所有模块都使用”。
- **预期结果**: dev log 覆盖主要生产者链路，并准确描述跨模块使用模式。
- **验证方式**: 代码检索 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-005: 下游消费与输出链路说明完整
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: `common/webui.go`、`common/utils.go`，开发者提交的新 dev log
- **前置条件**:
  - 可以读取输出相关源码与 dev log
- **测试步骤**:
  1. 阅读 `common/webui.go` 中的 `SortByValue`、`PrintSortedResults`、`PrintObjects`。
  2. 阅读 `common/utils.go` 中的 `PrintTextModeResults` 及 JSON 输出链路。
  3. 检查 dev log 是否说明 `ResultItems` 如何被排序、截断、渲染到 Web UI / HTTP / 文本输出。
  4. 检查 dev log 是否提到 Postman 或其它消费端复用一级结果结构的情况；如无，需明确实际范围。
- **预期结果**: dev log 能完整串起 `ResultItems` 从聚合结果到排序展示、文本输出、Web UI/HTTP 消费的主链路。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-006: 区分共享 ResultItem 与输出专用局部结构
- **关联 AC**: AC6
- **类型**: 边界
- **测试数据**: `common/result_items.go`、`common/utils.go`，开发者提交的新 dev log
- **前置条件**:
  - 可以读取源码与 dev log
- **测试步骤**:
  1. 确认 `common/result_items.go` 中存在共享的 `common.ResultItem`。
  2. 确认 `common/utils.go` 的 `PrintTextModeResults` 内定义了局部 `ResultItem`/`JsonResult` 输出结构。
  3. 检查 dev log 是否明确区分两者职责：前者用于运行期聚合，后者仅用于文本/JSON 序列化。
  4. 检查文档是否避免把输出局部结构误写成全局共享结果模型。
- **预期结果**: dev log 明确做出类型语义区分，避免命名相同导致的分析错误。
- **验证方式**: 代码对照 + 手动文档检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

### TC-007: 文档交付不引入不必要代码或测试资产
- **关联 AC**: AC7
- **类型**: 反向
- **测试数据**: 开发者提交内容
- **前置条件**:
  - 已获取本 issue 的全部变更列表
- **测试步骤**:
  1. 检查本 issue 变更文件列表。
  2. 确认没有新增功能实现代码修改来“顺便优化” `ResultItem`。
  3. 确认没有新增与 issue 无关的 `testdata/`、`validate.py`、`test_*.py` 或其它测试资产。
  4. 确认交付物主要为新的 dev log 文档。
- **预期结果**: 本 issue 保持文档分析任务边界，不引入额外功能代码或无必要测试实现。
- **验证方式**: 变更列表检查
- **验收命令**:
  ```bash
  无。该 issue 为文档分析任务，不要求 run_test.py 自动化验收。
  ```

## 结论

- 本 issue **不应新增可执行测试代码**。
- 本 issue 的 test case 以**文档覆盖性与分析准确性检查**为主。
- 后续执行验收时，应以代码对照和文档审阅为准，而不是运行 `testdata/run_test.py`。
