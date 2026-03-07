# Test Cases

## 关联计划
Feature: prim/maze#30 common ResultItem 及其上层使用分析文档

## 测试策略说明

本 issue 的交付物是 **dev log / 分析文档**，不是功能实现代码，也不要求 QA 执行运行时验收。因此这里设计的是**文档验收 test case**，重点验证：

1. 文档是否准确定位 `common.ResultItem` 的定义与职责
2. 文档是否覆盖主要上层调用链与输出链路
3. 文档是否区分运行态聚合结构与最终 JSON/text 输出结构
4. 文档是否给出足够的代码位置，便于开发者和后续维护者追踪

## Test Case 列表

### TC-001: 文档必须准确定位 common.ResultItem 的定义
- **关联 AC**: AC1（找到 common ResultItem）
- **类型**: 正向/文档准确性
- **测试数据**: 代码文件 `common/result_items.go`（现有）
- **前置条件**:
  - 已生成开发文档（新的 dev log）
  - 仓库工作区可读取源码
- **测试步骤**:
  1. 打开新的 dev log 文档。
  2. 检查文档是否明确指出 `common.ResultItem` 定义位于 `common/result_items.go`。
  3. 检查文档是否说明其核心字段：`Amount`、`Sizeof`、`Objects`、`ObjSize`。
  4. 对照源码确认字段和文件路径描述无误。
- **预期结果**:
  - 文档准确给出 `common/result_items.go` 中的 `ResultItem` 定义位置。
  - 文档对字段职责的描述与源码一致，无张冠李戴。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  python3 - <<'PY'
  from pathlib import Path
  text = Path('common/result_items.go').read_text()
  assert 'type ResultItem struct' in text
  assert 'Amount  uint64' in text
  assert 'Sizeof  uint64' in text
  assert 'Objects PriorityQueue' in text
  assert 'ObjSize map[uint64]uint64' in text
  print('TC-001 source anchor verified')
  PY
  ```

### TC-002: 文档必须覆盖 ResultItem 的核心操作函数
- **关联 AC**: AC1
- **类型**: 正向/文档完整性
- **测试数据**: `common/result_items.go`（现有）
- **前置条件**: 新的 dev log 已编写完成
- **测试步骤**:
  1. 阅读文档中关于 ResultItem 生命周期/操作流程的部分。
  2. 检查是否覆盖以下函数：`GetResultItem`、`AddToResultItems`、`MergeResultItems`、`DumpResultItems`。
  3. 对照源码确认文档对这些函数职责的说明是否正确。
- **预期结果**:
  - 文档明确说明 ResultItem 的创建/获取、累计、合并、落盘/输出等关键步骤。
  - 没有遗漏核心函数，也没有把无关函数误写成主链路。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  python3 - <<'PY'
  from pathlib import Path
  text = Path('common/result_items.go').read_text()
  for name in ['GetResultItem', 'AddToResultItems', 'MergeResultItems', 'DumpResultItems']:
      assert f'func {name}' in text, name
  print('TC-002 core functions verified')
  PY
  ```

### TC-003: 文档必须说明 ResultItem 的主要上层使用入口
- **关联 AC**: AC2（找到所有上层使用）
- **类型**: 正向/覆盖性
- **测试数据**: `python/maze.go`、`python/mobile_server2_main.go`、`python/mosd.go`、`txos/*.go`（现有）
- **前置条件**: 新的 dev log 已编写完成
- **测试步骤**:
  1. 检查文档是否至少覆盖以下上层模式：
     - 各模块内建立局部 `resultItems := make(map[uint64]*ResultItem)`
     - 通过闭包 `function := func(s uint64) *ResultItem { return GetResultItem(s, resultItems) }`
     - 扫描阶段调用 `AddToResultItems(...)`
     - 阶段结束调用 `MergeResultItems(resultItems)`
  2. 检查文档是否至少举出 Python 主流程和一个非 Python 模块（如 txos）的实际文件例子。
- **预期结果**:
  - 文档能够总结共性的“局部聚合 -> 全局合并”调用模式。
  - 文档至少给出 `python/maze.go` 和 `txos` 目录中的实际使用示例。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  python3 - <<'PY'
  from pathlib import Path
  files = ['python/maze.go', 'python/mobile_server2_main.go', 'python/mosd.go', 'txos/txos-aoi.go', 'txos/leak.go']
  hits = 0
  for f in files:
      text = Path(f).read_text()
      if 'GetResultItem(s, resultItems)' in text and 'MergeResultItems(resultItems)' in text:
          hits += 1
  assert hits >= 2, hits
  print('TC-003 upstream pattern verified, hits=', hits)
  PY
  ```

### TC-004: 文档必须区分运行态 common.ResultItem 与 JSON 输出结构
- **关联 AC**: AC3（编写介绍文档需准确描述上下游结构）
- **类型**: 边界/准确性
- **测试数据**: `common/result_items.go`、`common/utils.go`（现有）
- **前置条件**: 新的 dev log 已编写完成
- **测试步骤**:
  1. 阅读文档中关于“结果输出”或“最终展示”的章节。
  2. 检查文档是否明确说明：
     - `common/result_items.go` 中的 `ResultItem` 是运行态聚合结构；
     - `common/utils.go` 中存在用于 JSON 输出的局部结构（同名但不同语义）。
  3. 对照源码确认文档没有把两者混为一谈。
- **预期结果**:
  - 文档清晰区分运行时统计结构与序列化输出结构。
  - 文档指出 `common/utils.go` 的 JSON 输出结构字段如 `order/amount/total_size/avg_size/type/type_id`。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  python3 - <<'PY'
  from pathlib import Path
  r1 = Path('common/result_items.go').read_text()
  r2 = Path('common/utils.go').read_text()
  assert 'type ResultItem struct' in r1
  assert 'type ResultItem struct {' in r2
  assert 'json:"order"' in r2
  assert 'json:"type_id"' in r2
  print('TC-004 runtime vs json result structures verified')
  PY
  ```

### TC-005: 文档必须覆盖结果展示/消费链路
- **关联 AC**: AC2
- **类型**: 正向/完整性
- **测试数据**: `main.go`、`common/webui.go`、`common/utils.go`、`common/callback.go`（现有）
- **前置条件**: 新的 dev log 已编写完成
- **测试步骤**:
  1. 检查文档是否说明 `ResultItems` 不仅在采集阶段被写入，还会被多个展示/输出链路消费。
  2. 检查文档是否覆盖至少两种消费方式，例如：
     - text/JSON 输出；
     - web UI；
     - callback / topN 输出；
     - dump 到 LevelDB。
  3. 对照源码确认文档引用的位置真实存在。
- **预期结果**:
  - 文档能从“采集 -> 排序 -> 展示/导出”的角度说明上层使用，而不是只列扫描入口。
  - 文档给出消费链路对应代码文件。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  python3 - <<'PY'
  from pathlib import Path
  checks = {
      'main.go': 'SortByValue(ResultItems, SortBySize)',
      'common/webui.go': 'SortByValue(ResultItems, sortType)',
      'common/utils.go': 'SortByValue(ResultItems, SortBySize)',
      'common/callback.go': 'FormatResultType(order, s)',
  }
  for path, needle in checks.items():
      text = Path(path).read_text()
      assert needle in text, path
  print('TC-005 output consumption chain verified')
  PY
  ```

### TC-006: 文档必须识别并声明“所有上层使用”的边界范围
- **关联 AC**: AC2
- **类型**: 边界/范围定义
- **测试数据**: 全仓 `*.go`（现有）
- **前置条件**: 新的 dev log 已编写完成
- **测试步骤**:
  1. 检查文档是否说明“所有上层使用”的统计边界，例如：
     - 是否只统计当前主仓源码；
     - 是否排除 `sub-repos/lua-bak` 之类历史/备份目录；
     - 是否按“直接使用 ResultItem API”与“仅消费全局 ResultItems”分层描述。
  2. 对照代码搜索结果确认该边界定义合理。
- **预期结果**:
  - 文档不会模糊声称“全部覆盖”却不说明是否包含历史备份目录。
  - 文档能解释直接调用者与间接消费者的范围差异。
- **验证方式**: 手动代码检查
- **验收命令**:
  ```bash
  python3 - <<'PY'
  from pathlib import Path
  assert Path('sub-repos/lua-bak/sorter.go').exists()
  assert Path('common/result_items.go').exists()
  print('TC-006 repository scope anchors verified')
  PY
  ```

### TC-007: 文档应提供可复查的代码定位信息
- **关联 AC**: AC3
- **类型**: 正向/可维护性
- **测试数据**: 新 dev log 文档本身 + 相关源码（现有）
- **前置条件**: 新的 dev log 已编写完成
- **测试步骤**:
  1. 检查文档是否按模块列出关键文件路径。
  2. 检查文档是否包含足够的函数名/结构名，便于读者通过 grep 复查。
  3. 检查文档是否避免纯口头总结而缺少源码锚点。
- **预期结果**:
  - 文档至少包含以下源码锚点中的大部分：
    `common/result_items.go`、`common/utils.go`、`python/maze.go`、`common/webui.go`、`main.go`、`txos/*`。
  - 读者可根据文档内容直接定位到实现代码。
- **验证方式**: 文档审查
- **验收命令**:
  ```bash
  python3 - <<'PY'
  from pathlib import Path
  required = [
      'common/result_items.go',
      'common/utils.go',
      'python/maze.go',
      'common/webui.go',
      'main.go',
  ]
  for path in required:
      assert Path(path).exists(), path
  print('TC-007 document anchor files exist')
  PY
  ```

## 备注

- 本 issue 明确说明 **QA 不需要参与测试**，因此以上 test case 主要用于**文档交付验收标准设计**，不是要求新增 `testdata/` 或运行 `run_test.py` 的功能测试。
- 本 issue 也明确说明 **Dev 不需要编写代码**，因此本次不设计“功能失败即预期”的 TDD 测试文件；重点是确保后续提交的分析文档具备准确性、范围完整性和可追溯性。
