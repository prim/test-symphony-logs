# Test Cases

## 关联计划
Feature: 分析 common ResultItem 及其上层使用，并编写新的 dev log 介绍文档

## Test Case 列表

### TC-001: ResultItem 定义定位完整性检查
- **关联 AC**: AC1
- **类型**: 静态检查 / 文档验收
- **测试数据**: 仓库源码，重点关注 `common/result_items.go`
- **前置条件**:
  1. 已创建 `dev-flow/plan.md`
  2. 开发者已提交新的 dev log 文档
- **测试步骤**:
  1. 阅读 `common/result_items.go`
  2. 确认 dev log 明确说明 `ResultItem` 的定义位置
  3. 确认 dev log 覆盖以下结构要点：`Amount`、`Sizeof`、`Objects`、`ObjSize`
  4. 确认 dev log 提及至少这些辅助函数：`NewResultItem`、`GetResultItem`、`AddToResultItems`、`MergeResultItems`
- **预期结果**:
  - dev log 能准确指出 `ResultItem` 的定义文件与核心字段
  - dev log 对关键辅助函数职责说明准确，无明显遗漏
- **验证方式**: 手动代码审阅 + 文档比对
- **验收命令**:
  ```bash
  无自动化验收命令（文档分析任务）
  ```

### TC-002: 上层调用链覆盖性检查
- **关联 AC**: AC2
- **类型**: 静态检查 / 文档验收
- **测试数据**: 仓库源码，重点关注 `python/`、`nodejs/`、`mallocer/`、`txos/` 等目录
- **前置条件**:
  1. 开发者已提交新的 dev log 文档
- **测试步骤**:
  1. 搜索 `GetResultItem`、`AddToResultItems`、`MergeResultItems`、`AddResult` 的调用点
  2. 检查 dev log 是否覆盖主要上层模块：
     - Python 分析链路
     - Node.js 分析链路
     - mallocer/jemalloc 或 pymalloc 等聚合链路
     - txos 等业务/扩展链路
  3. 检查 dev log 是否说明典型模式：
     - 局部 `resultItems` 收集
     - `function := func(s uint64) *ResultItem { ... }` 适配
     - `MergeResultItems` 汇总到全局 `ResultItems`
     - `AddResult(do)` 作为统一封装入口
- **预期结果**:
  - dev log 对上层使用链路覆盖完整，至少覆盖主要模块与典型调用模式
  - 文档能帮助读者理解 ResultItem 从局部收集到全局汇总的流程
- **验证方式**: 手动代码审阅 + 文档比对
- **验收命令**:
  ```bash
  无自动化验收命令（文档分析任务）
  ```

### TC-003: 文档内容准确性与边界说明检查
- **关联 AC**: AC3
- **类型**: 文档质量 / 边界
- **测试数据**: 新增 dev log 文档 + 源码实现
- **前置条件**:
  1. 开发者已提交新的 dev log 文档
- **测试步骤**:
  1. 检查文档是否区分以下概念：
     - `common/result_items.go` 中真正的聚合结构 `ResultItem`
     - `common/utils.go` 中仅用于 JSON 输出的局部 `ResultItem` 结构
  2. 检查文档是否说明全局变量和并发相关点：
     - 全局 `ResultItems`
     - `ResultItemPool`
     - `MergeResultItems` 中的锁保护
  3. 检查文档是否说明输出/落盘相关能力：
     - `DumpResultItems`
     - text/json 输出阶段对结果的消费
  4. 检查文档是否避免错误结论，例如把文档任务描述成代码功能变更
- **预期结果**:
  - 文档表述准确，边界清晰
  - 不混淆运行时聚合结构与展示层输出结构
  - 不夸大、不误述实现行为
- **验证方式**: 手动代码审阅 + 文档比对
- **验收命令**:
  ```bash
  无自动化验收命令（文档分析任务）
  ```

## 说明

1. 本 issue 明确说明“QA 不需要参与测试”，因此本次不设计 `testdata/` 自动化用例，也不新增可执行测试代码。
2. 本次验收方式以**静态代码分析 + dev log 文档审阅**为主。
3. 本次重点不是运行期行为验证，而是确保分析文档覆盖准确、链路完整、边界清晰。

## Notes for Developer

- 建议 dev log 至少包含以下章节：
  - `ResultItem` 在哪里定义
  - 字段含义与数据结构选择
  - 局部收集 → 合并到全局的通用模式
  - Python / Node.js / mallocer / txos 的典型使用入口
  - `common/utils.go` 中同名展示结构与核心结构的区别
- 本 issue 是文档分析任务，不需要新增代码实现，也不需要新增自动化测试。
