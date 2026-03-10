# Test Cases

## 关联计划
Feature: 拆分 .maze.py — Phase 1: 无依赖部分

## Test File(s)
- `tests/test_issue_85_maze_py_split_phase1.py`: 针对 `.maze.py` Phase 1 拆分的结构、依赖和基础行为回归测试。

## Test Case 列表

### TC-001: 创建 Phase 1 拆分模块文件
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: 仓库根目录源码文件
- **前置条件**: 仓库工作区包含 `.maze.py`
- **测试步骤**:
  1. 检查仓库根目录是否存在 `maze_globals.py`
  2. 检查仓库根目录是否存在 `maze_utils.py`
  3. 检查仓库根目录是否存在 `maze_db.py`
- **预期结果**: 三个新文件都存在，且与 `.maze.py` 同目录，文件名前缀均为 `maze_`
- **验证方式**: 自动化测试 `test_phase1_modules_exist_at_repository_root`

### TC-002: `.maze.py` 按顺序导入拆分模块
- **关联 AC**: AC3, AC5
- **类型**: 正向
- **测试数据**: `.maze.py`
- **前置条件**: `.maze.py` 已完成 Phase 1 拆分
- **测试步骤**:
  1. 读取 `.maze.py`
  2. 检查是否包含 `from maze_globals import *`
  3. 检查是否包含 `from maze_utils import *`
  4. 检查是否包含 `from maze_db import *`
  5. 校验三者在文件中的出现顺序为 globals → utils → db
- **预期结果**: `.maze.py` 以指定顺序导入拆分模块
- **验证方式**: 自动化测试 `test_maze_py_imports_split_modules_in_required_order`

### TC-003: 已拆分定义从 `.maze.py` 移除，`is_mosd()` 保留
- **关联 AC**: AC5
- **类型**: 边界
- **测试数据**: `.maze.py`
- **前置条件**: 拆分只迁移无依赖部分
- **测试步骤**:
  1. 检查 `.maze.py` 中不再定义 `funcache/once/singleton/check_output/get_build_id/DBC/DB/CDB`
  2. 检查 `.maze.py` 中仍保留 `is_mosd()`
- **预期结果**: 仅无依赖定义被移出，依赖 `Process` 的 `is_mosd()` 继续保留
- **验证方式**: 自动化测试 `test_split_definitions_move_out_of_maze_py_but_is_mosd_stays`

### TC-004: `maze_globals.py` 保持装饰器与常量行为不变
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: `maze_globals.py`
- **前置条件**: `maze_globals.py` 可直接导入
- **测试步骤**:
  1. 导入 `maze_globals`
  2. 校验 `funcache` 仍然按参数缓存结果
  3. 校验 `once` 只执行一次目标方法
  4. 校验 `singleton` 返回实例而非类本身
  5. 校验 `MAX_POINTER` 常量值不变
- **预期结果**: 核心装饰器和常量语义与拆分前保持一致
- **验证方式**: 自动化测试 `test_maze_globals_preserves_core_decorator_behavior`

### TC-005: `maze_utils.py` 导出公共工具函数，且不迁移 `is_mosd()`
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: `maze_utils.py`
- **前置条件**: `maze_utils.py` 可导入
- **测试步骤**:
  1. 导入 `maze_utils`
  2. 校验存在 `log/fatal/run/check_output/get_build_id`
  3. 校验 `is_mosd` 不在 `maze_utils` 中
- **预期结果**: 工具函数成功迁移，`is_mosd()` 继续留在 `.maze.py`
- **验证方式**: 自动化测试 `test_maze_utils_exports_shared_helpers_without_is_mosd`

### TC-006: `maze_db.py` 导出 DBC/DB/CDB 且 DB 行为可回归
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: `maze_db.py`
- **前置条件**: `maze_db.py` 可导入
- **测试步骤**:
  1. 导入 `maze_db`
  2. 校验存在 `DBC`、`DB`、`CDB`
  3. 使用临时目录实例化 `DBC`
  4. mock `os.system` 后执行 `set/get`
  5. 校验版本匹配和不匹配时的返回结果
- **预期结果**: `DBC` 的核心持久化/读取行为不变
- **验证方式**: 自动化测试 `test_maze_db_exports_dbc_and_persistent_instances`

### TC-007: 新模块不引入新的外部依赖
- **关联 AC**: AC4
- **类型**: 兼容性
- **测试数据**: `maze_globals.py`, `maze_utils.py`, `maze_db.py`
- **前置条件**: 新模块已创建
- **测试步骤**:
  1. 解析三个新模块的 import 语句
  2. 校验仅使用标准库或内部模块 `maze_globals/maze_utils/maze_db`
- **预期结果**: 不引入任何新的第三方或外部依赖
- **验证方式**: 自动化测试 `test_split_modules_only_use_stdlib_or_internal_imports`

### TC-008: `.maze.py` 文件体量显著下降
- **关联 AC**: AC2
- **类型**: 性能
- **测试数据**: `.maze.py`
- **前置条件**: 已完成 Phase 1 拆分
- **测试步骤**:
  1. 统计 `.maze.py` 当前总行数
  2. 校验行数不高于 4500
- **预期结果**: `.maze.py` 相比拆分前显著缩减，满足“减少 ~500 行”的验收目标
- **验证方式**: 自动化测试 `test_maze_py_line_count_is_reduced_after_phase1_split`

## Cases
1. `test_phase1_modules_exist_at_repository_root`: 验证三个 Phase 1 新模块文件已创建。
2. `test_maze_py_imports_split_modules_in_required_order`: 验证 `.maze.py` 以规定顺序导入拆分模块。
3. `test_split_definitions_move_out_of_maze_py_but_is_mosd_stays`: 验证仅允许迁移的定义被拆出，`is_mosd()` 仍留在原文件。
4. `test_maze_globals_preserves_core_decorator_behavior`: 验证 globals 模块的装饰器和常量行为不变。
5. `test_maze_utils_exports_shared_helpers_without_is_mosd`: 验证 utils 模块导出工具函数且不包含 `is_mosd()`。
6. `test_maze_db_exports_dbc_and_persistent_instances`: 验证 db 模块的 `DBC/DB/CDB` 与持久化行为。
7. `test_split_modules_only_use_stdlib_or_internal_imports`: 验证无新增外部依赖。
8. `test_maze_py_line_count_is_reduced_after_phase1_split`: 验证 `.maze.py` 体量下降。

## Notes for Developer
- 这些测试是 TDD 基线，当前分支上预期失败，直到 Phase 1 拆分真正完成。
- 测试聚焦“结构保持 + 行为不变”的低风险重构目标，避免触发 `.maze.py` 运行期副作用。
- `is_mosd()` 被明确要求留在 `.maze.py`，不要一起迁移。
- 若新增 `maze_*.py` 文件，请保持与 `.maze.py` 同目录，且 import 顺序遵循 globals → utils → db。
