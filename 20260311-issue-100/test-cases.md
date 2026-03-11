# Test Cases

## 关联计划
Feature: 将 cpp-db 目录迁移至 3rd/misc/cpp-db

## Test Case 列表

### TC-001: 目录迁移到 3rd/misc
- **关联 AC**: AC1, AC2
- **类型**: 正向
- **测试数据**: 仓库目录结构
- **前置条件**: 仓库包含待迁移的 `cpp-db/` 目录与目标 `3rd/misc/` 目录
- **测试步骤**:
  1. 检查 `3rd/misc/cpp-db/` 目录是否存在
  2. 检查根目录 `cpp-db/` 是否已删除
  3. 检查 `uploader.py` 是否已随目录迁移到新位置
- **预期结果**: 新目录存在，旧目录不存在，迁移文件位于新位置
- **验证方式**: 自动化单元测试 `tests/test_issue_100_move_cpp_db.py::test_cpp_db_directory_moves_under_3rd_misc` 与 `test_uploader_moves_with_cpp_db_directory`

### TC-002: 运行时代码路径全部切换到新目录
- **关联 AC**: AC3, AC4
- **类型**: 正向
- **测试数据**: `maze_db.py`、`maze-py/cpp_analyzer.py`、`maze-py/messiahserver.py`、`gdb/gdb-scripts/az_netease_messiah.py`、`3rd/misc/query-cpp-db.py`
- **前置条件**: 代码库已完成路径引用修改
- **测试步骤**:
  1. 检查 `maze_db.py` 中 `CDB` 初始化路径
  2. 检查 C++ 分析器与相关脚本中的硬编码路径
  3. 审计上述运行时代码中是否还残留旧的 `cpp-db` 字面量
- **预期结果**: 所有运行时代码都引用 `3rd/misc/cpp-db`，不再保留旧路径
- **验证方式**: 自动化单元测试 `tests/test_issue_100_move_cpp_db.py::test_cpp_database_consumers_reference_new_runtime_paths` 与 `test_runtime_code_no_longer_uses_legacy_cpp_db_literals`

### TC-003: GDB 输出路径切换到新目录
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: `gdb/gdb-scripts/az_cpp.py`
- **前置条件**: GDB 脚本已完成迁移
- **测试步骤**:
  1. 检查 `ensure_gdb_output_dirs()` 中创建目录的路径
  2. 检查 `cpp.output`、`cpp.symbol_zero.output`、`cpp.stack_locals.output` 的输出路径
- **预期结果**: 所有输出文件均写入 `3rd/misc/cpp-db/`
- **验证方式**: 自动化单元测试 `tests/test_issue_100_move_cpp_db.py::test_az_cpp_uses_new_cpp_db_output_directory`

### TC-004: Release 脚本创建新的发布目录结构
- **关联 AC**: AC6
- **类型**: 正向
- **测试数据**: `maze`
- **前置条件**: 发布脚本已更新
- **测试步骤**:
  1. 检查 `maze` 脚本中 release 目录创建逻辑
  2. 确认存在 `release/3rd/misc/cpp-db`
  3. 确认旧的 `release/cpp-db` 创建逻辑已移除
- **预期结果**: 发布脚本创建新的层级目录结构，不再创建旧目录
- **验证方式**: 自动化单元测试 `tests/test_issue_100_move_cpp_db.py::test_release_script_creates_new_cpp_db_layout`

## Test File(s)
- `tests/test_issue_100_move_cpp_db.py`: 校验目录迁移、路径替换、GDB 输出路径与 release 目录结构
- `tests/test_issue_85_maze_py_split_phase1.py`: 同步更新旧断言，使其匹配迁移后的 `CDB` 路径

## Notes for Developer
- 测试采用静态断言方式，覆盖 issue 中列出的关键受影响文件与目录结构。
- 当前仓库仍保留旧目录和旧路径引用，因此新增测试现在失败是符合预期的。
- 若实现采用新的辅助常量，只要最终源码文本仍满足测试断言，或相应测试在后续重构中同步调整即可。
