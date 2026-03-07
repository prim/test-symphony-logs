# Test Cases

## 关联计划
Feature: prim/maze#31 — 分析 common ResultItem 及其上层使用，并编写新的 dev log 介绍

## 说明

该 issue 为**纯文档/分析任务**：

- 不要求新增功能代码
- 不要求新增自动化测试代码
- Issue 原文已明确：**QA 不需要参与测试**、**Dev 不需要编写代码**

因此本文件提供的是**文档验收型 test case**，用于评审开发者提交的分析文档是否完整、准确、可用于后续维护，而不是 `testdata/run_test.py` 形式的运行型测试。

## Test Case 列表

### TC-001: 准确定位 common.ResultItem 定义
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 仓库源码 `common/result_items.go`
- **前置条件**: 开发者已提交新的 dev log
- **测试步骤**:
  1. 阅读 dev log 中关于 `ResultItem` 定义位置的说明。
  2. 打开 `common/result_items.go`，确认文档是否指出 canonical 定义位置。
  3. 检查文档是否说明 `Amount`、`Sizeof`、`Objects`、`ObjSize` 的作用。
- **预期结果**:
  - 文档明确指出 `common/result_items.go` 中的 `type ResultItem struct` 是主定义。
  - 文档对核心字段和用途的解释与源码一致。
- **验证方式**: 代码检查 + 文档核对
- **验收命令**:
  ```bash
  无自动化命令，采用人工代码审阅
  ```

### TC-002: 覆盖核心辅助函数与数据流
- **关联 AC**: AC1, AC3
- **类型**: 正向
- **测试数据**: `common/result_items.go`、`common/webui.go`、`common/utils.go`
- **前置条件**: 开发者已提交新的 dev log
- **测试步骤**:
  1. 阅读 dev log 中 ResultItem 相关函数说明。
  2. 核对文档是否覆盖以下核心函数：`GetResultItem`、`AddToResultItems`、`AddToResultItems1`、`MergeResultItems`、`AddResult`。
  3. 核对文档是否说明排序/输出链路中的 `SortByValue` 与 `PrintTextModeResults`。
- **预期结果**:
  - 文档完整覆盖 ResultItem 的创建、累积、合并、排序、输出主链路。
  - 文档未将输出 DTO 与统计结构混为一谈。
- **验证方式**: 代码检查 + 文档核对
- **验收命令**:
  ```bash
  无自动化命令，采用人工代码审阅
  ```

### TC-003: 覆盖主要上层模块使用场景
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: `python/`、`txos/`、`mallocer/`、`nodejs/`、`postman/`
- **前置条件**: 开发者已提交新的 dev log
- **测试步骤**:
  1. 阅读 dev log 中“上层使用”章节。
  2. 检查文档是否至少覆盖以下模块中的主要使用方式：`python/`、`txos/`、`mallocer/`、`nodejs/`。
  3. 检查文档是否说明 `postman/` 更偏向结果消费/展示，而非结果生产。
  4. 对照源码中的代表性函数，确认文档示例不是虚构。
- **预期结果**:
  - 文档能按模块说明 ResultItem 的使用模式。
  - 每个主要模块至少给出一个代表性文件或函数。
  - 文档能够说明 dot-import 导致大量使用点表现为未限定名 `ResultItem`。
- **验证方式**: 代码检查 + 文档核对
- **验收命令**:
  ```bash
  无自动化命令，采用人工代码审阅
  ```

### TC-004: 区分同名结构，避免误导
- **关联 AC**: AC4
- **类型**: 边界
- **测试数据**: `common/utils.go`、`sub-repos/lua-bak/`
- **前置条件**: 开发者已提交新的 dev log
- **测试步骤**:
  1. 阅读 dev log 中关于“同名结构”的说明。
  2. 核对文档是否指出 `common/utils.go` 中 `PrintTextModeResults()` 内部的局部 `ResultItem` 仅为 JSON/text 输出 DTO。
  3. 核对文档是否指出 `sub-repos/lua-bak/` 中存在历史遗留的另一套 `ResultItem` 实现。
- **预期结果**:
  - 文档明确区分 canonical `common.ResultItem` 与同名局部/遗留结构。
  - 文档不会把这些结构错误归入主统计链路。
- **验证方式**: 代码检查 + 文档核对
- **验收命令**:
  ```bash
  无自动化命令，采用人工代码审阅
  ```

### TC-005: 文档结构满足 dev-log 规范
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: 新增 dev log 文档、`doc/development.md`
- **前置条件**: 开发者已提交新的 dev log
- **测试步骤**:
  1. 检查文件名是否符合 `dev-log/YYYY-MM-DD-<功能名称>.md`。
  2. 检查文档是否包含背景、解决方案/分析方法、主要内容、文件清单或等效结构。
  3. 检查文档是否具备可读的章节组织，而非仅堆砌 grep 结果。
- **预期结果**:
  - 文档命名与结构符合项目 dev-log 习惯。
  - 文档既有事实定位，也有总结性的分析结论。
- **验证方式**: 文档审阅
- **验收命令**:
  ```bash
  无自动化命令，采用人工文档审阅
  ```

### TC-006: 文档结论可支持后续维护者快速定位代码
- **关联 AC**: AC5
- **类型**: 反向
- **测试数据**: 新增 dev log 文档、对应源码文件
- **前置条件**: 开发者已提交新的 dev log
- **测试步骤**:
  1. 仅阅读 dev log，不预先浏览源码。
  2. 根据文档提供的路径与函数名，尝试定位 `ResultItem` 的定义、生产链路、合并链路、输出链路。
  3. 评估是否能在 10 分钟内完成核心代码跳转。
- **预期结果**:
  - 维护者可依据文档快速定位关键源码位置。
  - 文档中的模块分层与职责说明足够清晰，不会误导阅读顺序。
- **验证方式**: 人工可用性审阅
- **验收命令**:
  ```bash
  无自动化命令，采用人工文档审阅
  ```

## Notes for Developer

- 该 issue 的交付物本质上是**分析文档**，不是功能实现，也不是测试数据。
- 建议文档明确区分三类概念：
  1. **canonical 统计结构**：`common/result_items.go` 中的 `ResultItem`
  2. **结果排序/展示链路**：`common/webui.go`、`common/utils.go`
  3. **同名但不同用途的结构**：`common/utils.go` 局部 DTO、`sub-repos/lua-bak` 遗留实现
- 建议按“定义层 → 聚合层 → 模块调用层 → 输出层”的顺序编写，便于读者建立整体认知。
- 由于仓库大量使用 `import . "maze/common"`，文档应特别提醒：多数调用点不会写成 `common.ResultItem`，而是直接以 `ResultItem` / `AddToResultItems` / `MergeResultItems` 的未限定名出现。
