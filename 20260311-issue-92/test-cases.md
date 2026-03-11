# Test Cases

## 关联计划
Feature: 拆分 `python/classifier.go`

## Test File(s)
- `tests/test_issue_92_split_python_classifier.py`: 使用静态结构断言验证 `python/classifier.go` 已按职责拆分为 4 个文件，并确保关键符号落在正确文件中。

## Test Case 列表

### TC-001: 创建 4 个拆分目标文件
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 无需新增 `testdata`；检查源码布局
- **前置条件**: 代码分支已实现 issue #92 的文件拆分
- **测试步骤**:
  1. 检查 `python/` 目录下是否存在 `classifier.go`、`classifier_types.go`、`classifier_tree.go`、`classifier_objects.go`
  2. 检查 4 个文件是否都声明为 `package python`
- **预期结果**: 4 个文件全部存在，且仍归属于 `package python`
- **验证方式**: 自动化测试 `tests/test_issue_92_split_python_classifier.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-002: classifier.go 仅保留调度与共享状态
- **关联 AC**: AC2、AC6
- **类型**: 正向/边界
- **测试数据**: 无需新增 `testdata`；检查源码内容
- **前置条件**: 完成文件拆分
- **测试步骤**:
  1. 检查 `python/classifier.go` 是否保留 `ClassifyFunctionMap`、`dec(merge)`、`classifyPythonObject(...)`、`initPyClassify()`
  2. 检查该文件中不再包含 `ClassifyNodeSizeOfTree`、`ClassifyLambdaType`、`classifyListObject`、`classifyDictObject_3`、`IncludePyoRefcntOne`、`SizeofTree` 等已拆分逻辑
  3. 检查文件行数显著下降（阈值：≤ 550 行）
- **预期结果**: `classifier.go` 成为小型调度文件，不再承载对象实现和树分析细节
- **验证方式**: 自动化测试 `tests/test_issue_92_split_python_classifier.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-003: 类型特征结构移动到 classifier_types.go
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: 无需新增 `testdata`
- **前置条件**: 完成文件拆分
- **测试步骤**:
  1. 检查 `python/classifier_types.go` 是否包含 `ClassifyLambdaType`、`ClassifyArrayType`、`ClassifySimpleMapType`、`ClassifyMapType`、`ClassifyNeteaseType`、`ClassifyMb2ObjectType`、`TypeMapping`
  2. 检查该文件不包含 `ClassifyNodeSizeOfTree`、`SizeofTree`、`classifyPythonObject`、`classifyListObject`
- **预期结果**: 类型特征和映射结构集中在 `classifier_types.go`，且不混入其他职责
- **验证方式**: 自动化测试 `tests/test_issue_92_split_python_classifier.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-004: SizeofTree 相关逻辑移动到 classifier_tree.go
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: 无需新增 `testdata`
- **前置条件**: 完成文件拆分
- **测试步骤**:
  1. 检查 `python/classifier_tree.go` 是否包含 `ClassifyNodeSizeOfTree`、`AddNode`、`ToDot`、`ToString`、`SizeofTree`
  2. 检查该文件不包含 `ClassifyLambdaType`、`TypeMapping`、`classifyPythonObject`、`classifyListObject`、`initPyClassify`
- **预期结果**: 树遍历和 DOT 导出逻辑集中在 `classifier_tree.go`
- **验证方式**: 自动化测试 `tests/test_issue_92_split_python_classifier.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-005: 对象分类函数移动到 classifier_objects.go
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: 无需新增 `testdata`
- **前置条件**: 完成文件拆分
- **测试步骤**:
  1. 检查 `python/classifier_objects.go` 是否包含代表性的 `classify*Object` 函数（如 `classifyMethodObject`、`classifyCodeObject`、`classifyListObject`、`classifyDictObject_`、`classifyDictObject_3`）
  2. 检查该文件是否包含 `IncludePyoRefcntOne`、`classifySkip0`、`classifySkip1`
  3. 检查该文件不重新定义 `ClassifyLambdaType`、`ClassifyNodeSizeOfTree`、`classifyPythonObject`、`initPyClassify`
- **预期结果**: 具体对象解析和跳过/聚合辅助逻辑集中在 `classifier_objects.go`
- **验证方式**: 自动化测试 `tests/test_issue_92_split_python_classifier.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-006: 拆分后关键符号只有唯一实现
- **关联 AC**: AC2、AC3、AC4、AC5、AC6
- **类型**: 边界/错误处理
- **测试数据**: 无需新增 `testdata`
- **前置条件**: 完成文件拆分
- **测试步骤**:
  1. 在 4 个文件中统计 `ClassifyLambdaType`、`ClassifyNodeSizeOfTree`、`TypeMapping`、`SizeofTree`、`classifyMethodObject`、`classifyListObject`、`classifyDictObject_`、`classifyDictObject_3`、`IncludePyoRefcntOne`、`classifySkip0`、`classifyPythonObject`、`initPyClassify` 的定义次数
  2. 验证每个符号恰好定义 1 次
- **预期结果**: 不存在遗漏迁移、重复定义或新旧实现并存的情况
- **验证方式**: 自动化测试 `tests/test_issue_92_split_python_classifier.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-007: 代表性 Python 堆扫描回归验证
- **关联 AC**: AC7、AC8
- **类型**: 兼容性/回归
- **测试数据**: 复用 `testdata/python/20260129-complex-types-311`
- **前置条件**:
  1. 已完成代码拆分
  2. `testdata` 子模块已初始化
  3. 已执行 `./maze --build`
- **测试步骤**:
  1. 执行代表性 Python 堆扫描用例
  2. 观察 `maze-result.json` 生成情况
  3. 检查 `maze.log`、`maze.py.log` 中无异常崩溃或导入错误
- **预期结果**: 用例可通过，说明拆分未影响 Python 分类主流程
- **验证方式**: `run_test.py` + 日志检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

## Notes for Developer
- 本次新增的是 **TDD 结构性测试**，当前分支尚未拆分 `python/classifier.go`，因此新增测试预期会失败。
- 这些测试主要约束“文件边界”和“符号归属”，避免开发在拆分过程中出现“文件创建了，但旧实现仍留在原文件中”的半拆分状态。
- 完成拆分后，请补充执行 `./maze --build` 以及 `python3 testdata/run_test.py python/20260129-complex-types-311` 做运行时回归验证。
