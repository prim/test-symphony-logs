# Test Cases

## 关联计划
Feature: `prim/maze#35` — 分析 `common ResultItem` 及其上层使用，并编写新的 dev log 介绍。

## 说明

本 issue 的需求是**文档分析交付**，而不是功能实现或测试框架改动。原始需求已明确：

- QA 不需要参与测试
- Dev 不需要编写代码
- Dev 需要编写文档（分析报告）

因此本次不设计 `testdata/run_test.py` 形式的运行型验收用例，也**不新增自动化测试代码**。以下 test case 采用**文档审查 + 代码映射检查**方式，对 dev log 的完整性与准确性进行验收。

## Test Case 列表

### TC-001: ResultItem 定义与字段说明完整性
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 无；使用源码 `common/result_items.go`
- **前置条件**:
  - 已产出新的 dev log 文档
  - 工作区包含最新源码
- **测试步骤**:
  1. 阅读新 dev log。
  2. 检查文档是否明确指出 `common/result_items.go` 是 `ResultItem` 的定义位置。
  3. 检查文档是否逐项说明 `Amount`、`Sizeof`、`Objects`、`ObjSize` 的用途。
  4. 对照 `common/result_items.go` 中 `type ResultItem struct` 的实际字段定义确认无遗漏、无虚构字段。
- **预期结果**:
  - 文档准确给出 `ResultItem` 定义位置。
  - 文档完整说明 4 个核心字段的含义与职责。
- **验证方式**: 手动代码检查（文档与源码逐项对照）
- **验收命令**:
  ```bash
  # 本 TC 为文档审查，无 run_test.py 自动化命令
  ```

### TC-002: ResultItem 生命周期函数覆盖
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: 无；使用源码 `common/result_items.go`
- **前置条件**: 已产出新的 dev log 文档
- **测试步骤**:
  1. 阅读 dev log 的“主要改动”或“关键代码/分析”章节。
  2. 检查文档是否覆盖以下函数：`NewResultItem`、`GetResultItem`、`AddToResultItems`、`MergeResultItems`、`DumpResultItems`。
  3. 对照源码确认文档是否正确描述这些函数在创建、取用、累加、合并、落盘中的角色。
- **预期结果**:
  - 文档至少覆盖上述 5 个关键函数。
  - 描述与源码实现语义一致，无明显错误。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  # 本 TC 为文档审查，无 run_test.py 自动化命令
  ```

### TC-003: WebUI 一级/二级结果链路说明
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: 无；使用源码 `common/webui.go`
- **前置条件**: 已产出新的 dev log 文档
- **测试步骤**:
  1. 阅读 dev log 中关于结果展示链路的章节。
  2. 检查文档是否说明 `SortByValue`、`PrintSortedResults`、`PrintObjects` 的职责。
  3. 检查文档是否提到 `SortedResultMap` 如何将一级表格 order 映射回 class/type。
  4. 检查文档是否提到 `GlobalS` 在对象二级展示过程中的作用。
  5. 对照 `common/webui.go` 源码核实说明是否准确。
- **预期结果**:
  - 文档能说明 `ResultItem` 从聚合结果到 WebUI 一级/二级展示的关键路径。
  - 文档包含 `SortedResultMap` 与 `GlobalS` 的说明。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  # 本 TC 为文档审查，无 run_test.py 自动化命令
  ```

### TC-004: 上层回调扩展点说明
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: 无；使用源码 `common/callback.go`
- **前置条件**: 已产出新的 dev log 文档
- **测试步骤**:
  1. 阅读 dev log。
  2. 检查文档是否说明 `common/callback.go` 提供的回调扩展点。
  3. 检查文档是否明确提到 `FormatResultType`、`PrintObjectsNormal`、`SamplePostmanObjects` 等变量可被上层模块覆写。
  4. 对照源码确认描述准确。
- **预期结果**:
  - 文档解释公共层和语言层之间的解耦方式。
  - 文档能帮助读者理解为什么不同语言可以复用 `ResultItem` 但定制展示逻辑。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  # 本 TC 为文档审查，无 run_test.py 自动化命令
  ```

### TC-005: Python 接入点覆盖
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: 无；使用源码 `python/maze.go`、`python/results.go`
- **前置条件**: 已产出新的 dev log 文档
- **测试步骤**:
  1. 阅读 dev log。
  2. 检查文档是否提到 Python 模块通过 `GetResultItem` / `AddToResultItems` / `MergeResultItems` 写入结果。
  3. 检查文档是否提到 Python 模块在 `initMazeCallback()` 中绑定 `FormatResultType`、`PrintObjectsNormal`、`SamplePostmanObjects`。
  4. 对照 `python/maze.go`、`python/results.go` 确认描述正确。
- **预期结果**:
  - 文档准确说明 Python 上层如何接入 `ResultItem` 数据结构和展示回调。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  # 本 TC 为文档审查，无 run_test.py 自动化命令
  ```

### TC-006: Node.js 接入点覆盖
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: 无；使用源码 `nodejs/maze.go`
- **前置条件**: 已产出新的 dev log 文档
- **测试步骤**:
  1. 阅读 dev log。
  2. 检查文档是否提到 Node.js 通过 `main(function func(s uint64) *ResultItem)` 写入结果。
  3. 检查文档是否提到 Node.js 绑定 `FormatResultType`、`PrintObjectsNormal`、`ShowGraphG` 等回调。
  4. 对照 `nodejs/maze.go` 确认描述准确。
- **预期结果**:
  - 文档覆盖 Node.js 模块与 `ResultItem` 的接入方式。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  # 本 TC 为文档审查，无 run_test.py 自动化命令
  ```

### TC-007: Txos 接入点覆盖
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: 无；使用源码 `txos/maze.go`、`txos/results.go` 及相关调用点
- **前置条件**: 已产出新的 dev log 文档
- **测试步骤**:
  1. 阅读 dev log。
  2. 检查文档是否说明 Txos 大量通过 `AddToResultItems` / `tryMergeAddToResultItems` 写入结果。
  3. 检查文档是否说明 Txos 绑定 `FormatResultType`、`PrintObjectsNormal`、`SamplePostmanObjects`。
  4. 对照 `txos/maze.go`、`txos/results.go` 及典型分类文件确认描述成立。
- **预期结果**:
  - 文档覆盖 Txos 作为上层使用者的写入与展示路径。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  # 本 TC 为文档审查，无 run_test.py 自动化命令
  ```

### TC-008: 输出链路完整性说明
- **关联 AC**: AC6
- **类型**: 正向
- **测试数据**: 无；使用源码 `common/utils.go`、`common/webui.go`、`postman/postman-profile-main.go`
- **前置条件**: 已产出新的 dev log 文档
- **测试步骤**:
  1. 阅读 dev log 中关于输出结果的章节。
  2. 检查文档是否说明文本模式如何基于 `ResultItems` 排序并输出结果。
  3. 检查文档是否说明 WebUI 与 Postman 如何分别消费一级/二级结果。
  4. 对照 `common/utils.go`、`common/webui.go`、`postman/postman-profile-main.go` 确认描述准确。
- **预期结果**:
  - 文档能让读者理解 `ResultItem` 与 text / webui / postman 三类输出之间的关系。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  # 本 TC 为文档审查，无 run_test.py 自动化命令
  ```

### TC-009: 非功能性交付边界说明
- **关联 AC**: AC7
- **类型**: 边界
- **测试数据**: Git diff、dev log
- **前置条件**: 开发者已提交本 issue 的交付内容
- **测试步骤**:
  1. 检查 Git diff。
  2. 确认本 issue 交付以 dev log / 文档为主。
  3. 确认没有与本 issue 无关的功能实现代码改动；如有少量辅助性改动，需在文档中给出合理解释。
- **预期结果**:
  - 交付边界与需求一致，以分析文档为核心。
- **验证方式**: 代码审查 / Git diff 检查
- **验收命令**:
  ```bash
  git diff --stat
  ```

## Notes for Developer

1. 本 issue 的核心是**把源码事实整理成高可读的分析文档**，不是补测试、不是补功能。
2. 建议 dev log 重点围绕“数据流”组织内容，而不是只罗列文件：
   - `ResultItem` 如何创建
   - 如何被上层累计写入
   - 如何被公共层排序/截断/映射
   - 如何被不同前端（text/webui/postman）消费
3. 建议在 dev log 中至少引用这些文件：
   - `common/result_items.go`
   - `common/webui.go`
   - `common/callback.go`
   - `common/utils.go`
   - `python/maze.go`, `python/results.go`
   - `nodejs/maze.go`
   - `txos/maze.go`, `txos/results.go`
   - `postman/postman-profile-main.go`
4. 建议 dev log 明确区分：
   - **公共数据结构层**
   - **公共展示层**
   - **语言/业务模块适配层**
5. 本次未创建自动化测试文件，原因是需求本身明确排除了 QA 测试工作，且交付物是分析文档，不是可运行功能。
