# Test Cases

## 关联计划
Feature: 分析 common ResultItem 及其上层使用 (A)

## 说明

本任务是**文档/分析任务**，根据 `dev-flow/plan.md`：

- 不要求新增功能代码
- 不要求新增测试代码
- 交付物是新的 `dev-log` 分析文档

因此本次不设计 `testdata/` 下的自动化运行用例，也不创建 `test_*.py`。验收方式以**代码检查 + 文档检查**为主，确认分析范围完整、引用准确、链路清晰。

## Test Case 列表

### TC-001: ResultItem 定义定位完整性检查
- **关联 AC**: AC1
- **类型**: 文档检查 / 静态检查
- **测试数据**: 现有源码 `common/result_items.go`
- **前置条件**:
  1. 已产出新的 `dev-log` 分析文档
- **测试步骤**:
  1. 阅读新 dev log
  2. 核对文档是否明确指出 `common.ResultItem` 的定义位置
  3. 核对文档是否解释以下字段含义：`Amount`、`Sizeof`、`Objects`、`ObjSize`
- **预期结果**:
  - 文档准确指出 `ResultItem` 定义于 `common/result_items.go`
  - 文档对核心字段的说明与源码一致
- **验证方式**: 手动比对文档与源码
- **验收命令**:
  ```bash
  grep -n "type ResultItem struct" common/result_items.go
  ```

### TC-002: ResultItem 核心 API 覆盖检查
- **关联 AC**: AC2
- **类型**: 文档检查 / 静态检查
- **测试数据**: 现有源码 `common/result_items.go`
- **前置条件**:
  1. 已产出新的 `dev-log` 分析文档
- **测试步骤**:
  1. 阅读新 dev log
  2. 检查文档是否覆盖并解释以下 API：
     - `NewResultItem`
     - `GetResultItem`
     - `AddToResultItems`
     - `MergeResultItems`
     - `AddResult`
  3. 核对描述与源码实现职责是否一致
- **预期结果**:
  - 文档包含上述全部 API
  - 每个 API 的职责描述清晰且无明显误导
- **验证方式**: 手动比对文档与源码
- **验收命令**:
  ```bash
  grep -nE "func (NewResultItem|GetResultItem|AddToResultItems|MergeResultItems|AddResult)" common/result_items.go
  ```

### TC-003: 上层使用模块覆盖检查
- **关联 AC**: AC3
- **类型**: 文档检查 / 静态检查
- **测试数据**: 现有源码目录
- **前置条件**:
  1. 已产出新的 `dev-log` 分析文档
- **测试步骤**:
  1. 阅读新 dev log
  2. 检查文档是否分组说明以下模块对 `ResultItem` 的主要上层使用：
     - `mallocer/`
     - `python/`
     - `cpp/`
     - `lua/`
     - `txos/`
     - `common/`
  3. 对照源码，确认文档至少列出每个子系统中的代表性调用点或调用模式
- **预期结果**:
  - 文档覆盖所有要求子系统
  - 每个子系统至少有一个真实源码位置或调用模式说明
- **验证方式**: 手动比对文档与源码搜索结果
- **验收命令**:
  ```bash
  rg "GetResultItem\(|AddToResultItems\(|MergeResultItems\(|AddResult\(" common mallocer python cpp lua txos
  ```

### TC-004: 局部收集到全局合并链路说明检查
- **关联 AC**: AC4
- **类型**: 文档检查 / 设计链路检查
- **测试数据**: `common/result_items.go`、`python/maze.go`、`mallocer/pieces.go` 等现有源码
- **前置条件**:
  1. 已产出新的 `dev-log` 分析文档
- **测试步骤**:
  1. 阅读新 dev log
  2. 检查文档是否明确描述以下链路：
     - 创建局部 `resultItems`
     - 通过 `GetResultItem` / `AddToResultItems` 收集局部结果
     - 通过 `MergeResultItems` 汇总到全局 `ResultItems`
     - 通过排序/格式化逻辑进行输出
  3. 核对文档中是否解释 `AddResult` 作为常见包装模式的作用
- **预期结果**:
  - 文档包含完整结果流转链路
  - 文档能解释并发/分片场景下为何使用局部 map + merge
- **验证方式**: 手动比对文档与源码
- **验收命令**:
  ```bash
  rg "make\(map\[uint64\]\*ResultItem\)|MergeResultItems\(|AddResult\(" common mallocer python cpp lua txos
  ```

### TC-005: 输出与消费链路说明检查
- **关联 AC**: AC4
- **类型**: 文档检查 / 静态检查
- **测试数据**: `common/utils.go`、`common/webui.go`、`http.go`、`python/results.go`、`txos/results.go`
- **前置条件**:
  1. 已产出新的 `dev-log` 分析文档
- **测试步骤**:
  1. 阅读新 dev log
  2. 检查文档是否说明 `ResultItems` 在输出阶段的主要消费方式，包括：
     - 排序
     - text/json 输出
     - Web/API 展示
     - 二级对象查看
  3. 核对文档中引用的文件和职责是否准确
- **预期结果**:
  - 文档覆盖主要输出/消费位置
  - 文档未把“收集逻辑”和“展示逻辑”混淆
- **验证方式**: 手动比对文档与源码
- **验收命令**:
  ```bash
  rg "SortByValue\(|PrintSortedResults\(|maze-result.json|ResultItems\[" common http.go python txos
  ```

### TC-006: Dev log 交付规范检查
- **关联 AC**: AC5
- **类型**: 文档规范检查
- **测试数据**: 新增 `dev-log/YYYY-MM-DD-*.md`
- **前置条件**:
  1. 已产出新的 `dev-log` 分析文档
- **测试步骤**:
  1. 检查文件是否放在 `dev-log/` 目录下
  2. 检查文件命名是否符合 `YYYY-MM-DD-<功能名称>.md`
  3. 检查文档是否包含基本结构：背景、分析内容/解决方案、关键位置、文件清单或类似结构
- **预期结果**:
  - 文档路径和命名符合规范
  - 文档结构完整，适合后续开发者阅读
- **验证方式**: 手动检查
- **验收命令**:
  ```bash
  ls dev-log/*.md
  ```

### TC-007: 文档任务边界符合性检查
- **关联 AC**: AC6
- **类型**: 回归 / 范围控制
- **测试数据**: git 变更集
- **前置条件**:
  1. 已完成本次 issue 交付
- **测试步骤**:
  1. 检查本次变更是否仅包含文档类文件（尤其是新的 `dev-log`）以及本地工作文件
  2. 确认未引入功能实现代码修改
  3. 确认未新增 `testdata/` 自动化测试文件
- **预期结果**:
  - 变更范围符合“仅分析文档，不改功能代码、不写测试代码”的要求
- **验证方式**: 查看 git diff / git status
- **验收命令**:
  ```bash
  git diff --stat
  ```

## Notes for Developer

- 本 issue 的核心不是功能正确性，而是**分析范围完整性**和**文档准确性**。
- 建议 dev log 重点说明统一模式：
  1. 局部 `resultItems` map 创建
  2. `GetResultItem` 获取/复用条目
  3. `AddToResultItems` 累加统计
  4. `MergeResultItems` 合并到全局 `ResultItems`
  5. `SortByValue` / text/json/web 输出消费结果
- 建议按子系统分组列出调用点，避免只罗列 grep 结果，提升可读性。
- 建议特别说明 `ResultItem` 不只是计数结构，也承载二级对象采样数据（`Objects` / `ObjSize`）。
