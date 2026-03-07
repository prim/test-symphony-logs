# Test Cases

## 关联计划
Feature: 分析 common ResultItem 及其上层使用 (B)

## Test Case 列表

### TC-001: 定位 ResultItem 定义与字段职责
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 现有源码 `common/result_items.go`
- **前置条件**:
  - 工作区包含 issue #42 的最新代码
  - 可读取 `dev-flow/plan.md`
- **测试步骤**:
  1. 阅读 `common/result_items.go`。
  2. 确认文档明确指出 `ResultItem` 的定义位置。
  3. 确认文档解释 `Amount`、`Sizeof`、`Objects`、`ObjSize` 四个字段的职责。
  4. 确认文档说明 `PriorityQueue`、`ResultItemPool` 与 `ResultItem` 的关系。
- **预期结果**:
  - 新 dev log 能准确说明 `ResultItem` 的结构、字段语义、对象采样容器与池化复用机制。
- **验证方式**: 代码检查 + 文档检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```
  注意：本 TC 的核心验收以代码/文档审查为主；命令仅用于遵循统一验收入口格式，不作为是否通过的唯一依据。

### TC-002: 核心接口覆盖完整
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 现有源码 `common/result_items.go`、`common/webui.go`
- **前置条件**:
  - 已有一篇新增 dev log 草稿或成稿
- **测试步骤**:
  1. 检查文档是否覆盖以下接口：`NewResultItem`、`GetResultItem`、`AddToResultItems`、`AddToResultItems1`、`MergeResultItems`、`SortByValue`、`DumpResultItems`/`DumpResultItems_`、`AddResult`。
  2. 核对每个接口的职责描述是否与源码一致。
  3. 检查是否说明 `BigSizeTypeMem` 与大对象类型重映射逻辑。
- **预期结果**:
  - 文档完整覆盖 ResultItem 的创建、获取、累加、合并、排序、导出、包装调用链。
- **验证方式**: 代码检查 + 文档检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```
  注意：验收命令仅作为统一入口占位，本 TC 以静态审查为准。

### TC-003: Python 路径的上层使用被正确分析
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: 现有源码 `python/maze.go`、`python/mosd.go`、`python/messiah_classify_h42.go`、`python/mos-classify.go`
- **前置条件**:
  - 能访问 Python 分析相关源码
- **测试步骤**:
  1. 阅读文档中关于 Python 路径的章节。
  2. 核对文档是否说明：页内/局部 `resultItems` 聚合、`GetResultItem` 写入、`MergeResultItems` 汇总到全局 `ResultItems`。
  3. 核对文档是否覆盖 Python 主流程和至少一个分类扩展路径（如 `mosd`/`messiah`）。
- **预期结果**:
  - 文档能说明 Python 对象分析如何通过局部 map + 合并机制使用 `ResultItem`。
- **验证方式**: 代码检查 + 文档检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-004: Node.js 路径的上层使用被正确分析
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: 现有源码 `nodejs/maze.go`
- **前置条件**:
  - 能访问 Node.js 分析相关源码
- **测试步骤**:
  1. 检查文档是否说明 `nodejs.Main()` 中通过 `AddResult(do)` 包装主扫描流程。
  2. 核对文档是否指出 `main(function)` 通过 `function func(s uint64) *ResultItem` 将扫描结果落入 `ResultItem`。
  3. 核对文档是否描述 HeapWalk 成功路径与 fallback 扫描路径都复用相同聚合接口。
- **预期结果**:
  - 文档准确覆盖 Node.js 结果采集如何接入 `ResultItem` 聚合层。
- **验证方式**: 代码检查 + 文档检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py nodejs/20260211-comprehensive
  ```

### TC-005: malloc / memory piece 路径的上层使用被正确分析
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: 现有源码 `mallocer/pieces.go`、`mallocer/pymalloc/pymalloc.go`、`mallocer/jemalloc/*.go`
- **前置条件**:
  - 能访问 mallocer 相关源码
- **测试步骤**:
  1. 检查文档是否说明 `InitAllMemoryPieceSize()` 会重置全局 `ResultItems`。
  2. 核对文档是否说明 `initAllMemoryPieceSize()`、pymalloc/jemalloc 等路径通过 `AddToResultItems` 或 `AddResult` 归集结果。
  3. 核对文档是否描述 ResultItem 在“内存块分类结果容器”中的角色，而不仅限于语言对象统计。
- **预期结果**:
  - 文档准确覆盖内存块分类、内存池遍历等上层模块对 `ResultItem` 的使用方式。
- **验证方式**: 代码检查 + 文档检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260201-basic-malloc
  ```

### TC-006: 输出链路的上层使用被正确分析
- **关联 AC**: AC2, AC3
- **类型**: 正向
- **测试数据**: 现有源码 `common/webui.go`、`common/utils.go`、`http.go`、`postman/postman-profile-main.go`、`txos/results.go`
- **前置条件**:
  - 能访问输出层相关源码
- **测试步骤**:
  1. 检查文档是否说明 `SortByValue`、`PrintSortedResults`、`PrintObjects` 如何消费 `ResultItems`。
  2. 核对文档是否覆盖 text/json 输出、HTTP L0/L1 输出、Postman 输出三类结果消费方式。
  3. 核对文档是否说明 `Objects` / `ObjSize` 在二级对象展示中的作用。
- **预期结果**:
  - 文档完整说明 `ResultItem` 如何从聚合结果进入文本、JSON、HTTP、Postman 等展示层。
- **验证方式**: 代码检查 + 文档检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py nodejs/20260225-maze-vs-heapsnapshot
  ```

### TC-007: 新 dev log 文件满足项目规范
- **关联 AC**: AC3, AC5
- **类型**: 边界
- **测试数据**: 新增 `dev-log/YYYY-MM-DD-*.md`
- **前置条件**:
  - 开发者已提交新的分析文档
- **测试步骤**:
  1. 检查文件路径是否位于 `dev-log/`。
  2. 检查文件命名是否符合 `YYYY-MM-DD-<name>.md`。
  3. 检查文档结构是否包含背景、分析/解决方案、主要内容、文件清单或同等层次结构。
  4. 检查标题与正文是否明确说明是对 `common ResultItem` 及其上层使用的分析。
- **预期结果**:
  - 文档命名与结构符合项目 dev-log 规范，可作为后续维护资料。
- **验证方式**: 文档检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```
  注意：本 TC 为文档规范校验，验收以人工检查为准。

### TC-008: 文档不越界到功能实现或测试实现
- **关联 AC**: AC4
- **类型**: 反向
- **测试数据**: git diff、开发报告、新增 dev log
- **前置条件**:
  - 本 issue 的变更已提交到当前工作区
- **测试步骤**:
  1. 检查变更集中是否存在功能实现代码修改（如 `common/`、`python/`、`nodejs/`、`mallocer/` 等业务逻辑变更）。
  2. 检查是否新增与本 issue 无关的测试逻辑或功能分支。
  3. 核对开发报告与 dev log 是否都将交付物描述为“分析文档”。
- **预期结果**:
  - 该 issue 的交付以分析文档为主，不包含额外功能实现。
- **验证方式**: 代码检查 + git diff 审查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```
  注意：本 TC 的核心依据是 diff 审查，而非自动化结果。

## 补充说明

- 本 issue 的需求本质是“代码分析文档交付”，不是功能新增，因此 test case 以**文档审查 + 源码对照**为主。
- 按项目统一格式保留了 `run_test.py` 验收命令字段，但这些命令不构成本 issue 的唯一验收依据。
- 本次 QA 设计阶段**不创建新的 testdata，也不编写功能测试代码**，因为需求中已明确“QA 不需要参与测试 / Dev 不需要编写代码”。
