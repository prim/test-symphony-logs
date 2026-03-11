# Test Cases

## 关联计划
Feature: 将类拆分至 maze-py 文件夹并使用 Builtins 全局注入

## Test File(s)
- `tests/test_issue_89_extract_classes_to_maze_py.py`: 验证 `.maze.py` 的 builtins 注入方案、`maze-py/` 目录拆分结果、模块导入约束与单例加载行为。

## Test Case 列表

### TC-001: 创建 maze-py 目录及目标模块文件
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: 仓库源码树
- **前置条件**: 开发者已按计划完成目录与文件拆分
- **测试步骤**:
  1. 检查项目根目录是否存在 `maze-py/`
  2. 检查 `maze-py/` 下是否存在 `mosd.py`、`asiocore.py`、`messiahserver.py`、`txos.py`、`g78.py`、`nodejs.py`、`taggeddict.py`
- **预期结果**: `maze-py/` 与全部目标模块文件存在
- **验证方式**: 自动化测试 `test_maze_py_directory_and_expected_modules_exist`

### TC-002: builtins 兼容导入与公共符号挂载
- **关联 AC**: AC4
- **类型**: 正向/兼容性
- **测试数据**: `.maze.py`
- **前置条件**: `.maze.py` 已接入 builtins 注入方案
- **测试步骤**:
  1. 检查 `.maze.py` 是否包含 `builtins` / `__builtin__` 兼容导入逻辑
  2. 检查 `singleton` 是否将实例注册到 `builtins`
  3. 检查 `once`、`singleton`、`log`、`run`、`fatal`、`Process` 是否显式挂载到 `builtins`
- **预期结果**: 兼容导入与公共符号挂载逻辑齐全
- **验证方式**: 自动化测试 `test_maze_py_bootstraps_builtins_and_registers_shared_helpers`

### TC-003: .maze.py 改为导入拆分模块且删除内联类实现
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: `.maze.py`
- **前置条件**: 目标类已从 `.maze.py` 中迁移出去
- **测试步骤**:
  1. 检查 `.maze.py` 是否将 `maze-py/` 加入模块搜索路径
  2. 检查 `.maze.py` 是否显式导入各拆分模块
  3. 检查 `.maze.py` 中是否已删除 `Mosd`、`Asiocore`、`AsiocoreNew`、`MessiahServer`、`Txos`、`G78`、`Nodejs`、`Taggeddict` 的类定义
- **预期结果**: `.maze.py` 只保留导入入口，不再内联这些类实现
- **验证方式**: 自动化测试 `test_maze_py_imports_split_modules_and_removes_inline_class_definitions`

### TC-004: 拆分模块不显式导入基础运行时依赖
- **关联 AC**: AC3
- **类型**: 反向/兼容性
- **测试数据**: `maze-py/*.py`
- **前置条件**: 各模块已落盘
- **测试步骤**:
  1. 解析每个拆分模块的 AST
  2. 检查是否显式导入 `builtins`、`__builtin__`、`maze_globals`、`maze_utils`、`maze_db`、`maze_collect`
  3. 检查是否使用 `from ... import singleton/once/log/run/fatal/Process/GDB/Language/MAX_POINTER`
- **预期结果**: 不存在上述显式基础依赖导入
- **验证方式**: 自动化测试 `test_split_modules_do_not_import_core_runtime_dependencies_explicitly`

### TC-005: 仅依赖 builtins 注入即可导入拆分模块
- **关联 AC**: AC3, AC4
- **类型**: 正向/兼容性
- **测试数据**: `maze-py/*.py`
- **前置条件**: 模块源码保持 `@singleton` / `@once` 原样使用
- **测试步骤**:
  1. 在测试进程的 `builtins` 中注入假的 `singleton`、`once`、`log`、`run`、`fatal`、`Process`、`GDB` 等符号
  2. 逐个动态加载拆分模块
  3. 断言模块可成功导入，且目标类名被注册为单例实例
- **预期结果**: 模块在无显式基础导入的情况下仍可通过 builtins 注入完成加载
- **验证方式**: 自动化测试 `test_split_modules_can_be_imported_with_builtins_injection_only`

### TC-006: .maze.py 体积显著缩小
- **关联 AC**: AC1
- **类型**: 边界
- **测试数据**: `.maze.py`
- **前置条件**: 完成类拆分
- **测试步骤**:
  1. 统计 `.maze.py` 总行数
  2. 校验行数是否降到 4000 行及以下
- **预期结果**: `.maze.py` 不再维持超大单文件规模
- **验证方式**: 自动化测试 `test_maze_py_file_size_is_reduced_after_extraction`

## Notes for Developer
- 新增测试遵循仓库现有 `tests/` 目录风格，主要做结构与导入约束验证，不触发真正的 GDB 分析流程。
- `test_split_modules_can_be_imported_with_builtins_injection_only` 会通过伪造 `builtins` 环境验证“作用域错觉”是否成立，因此拆分模块不应依赖显式 `import singleton` 或 `from maze_utils import log`。
- 行数阈值设置为 4000，是为了验证 `.maze.py` 发生了实质性瘦身，而不是仅做微小移动。
- 这些测试在当前状态下预期失败，属于 TDD 的正确结果。
