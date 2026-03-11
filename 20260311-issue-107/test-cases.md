# Test Cases

## 关联计划
Feature: prim/maze#107 统一项目目录结构重构 - 整合 cpp-db、maze-db、prim-gdb 迁移

## Test File(s)
- `tests/test_issue_107_directory_migration.py`：覆盖目录迁移、路径重写、release 打包与 submodule 配置校验

## Test Case 列表

### TC-001: 三个目标目录迁移到新位置
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 仓库目录结构
- **前置条件**: 完成目录迁移实现
- **测试步骤**:
  1. 检查 `3rd/misc/cpp-db/` 是否存在
  2. 检查 `gdb/maze-db/` 是否存在
  3. 检查 `gdb/prim-gdb/` 是否存在
- **预期结果**: 三个目录均存在于新位置
- **验证方式**: 自动化测试 `tests/test_issue_107_directory_migration.py::test_target_directory_layout_rehomes_cpp_db_maze_db_and_prim_gdb`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-002: 根目录旧目录被彻底移除
- **关联 AC**: AC2
- **类型**: 反向
- **测试数据**: 仓库目录结构
- **前置条件**: 完成目录迁移实现
- **测试步骤**:
  1. 检查根目录是否还存在 `cpp-db/`
  2. 检查根目录是否还存在 `maze-db/`
  3. 检查根目录是否还存在 `prim-gdb/`
- **预期结果**: 根目录不再保留旧目录
- **验证方式**: 自动化测试 `test_legacy_root_directories_are_removed_after_migration`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-003: 核心模块与 GDB 脚本全部改用新路径
- **关联 AC**: AC3
- **类型**: 正向/边界
- **测试数据**: `maze-py/maze_db.py`、`maze-py/gdb_wrapper.py`、`maze-py/cpp_analyzer.py`、`maze-py/messiahserver.py`、`gdb/gdb-scripts/*.py`
- **前置条件**: 完成路径更新
- **测试步骤**:
  1. 校验 `maze-py/maze_db.py` 中 DB/CDB 初始化路径
  2. 校验 GDB wrapper 使用 `gdb/prim-gdb` 路径
  3. 校验 C++ 分析器和 GDB 脚本全部写入新缓存目录
  4. 校验相关文件不再出现旧路径字面量
- **预期结果**: 所有运行时代码仅引用新路径
- **验证方式**: 自动化测试 `test_maze_db_module_uses_new_cache_locations` 与 `test_runtime_and_gdb_scripts_stop_referencing_legacy_locations`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-004: .gitmodules 正确迁移 prim-gdb submodule
- **关联 AC**: AC4
- **类型**: 兼容性
- **测试数据**: `.gitmodules`
- **前置条件**: submodule 配置已更新
- **测试步骤**:
  1. 检查 submodule 名称是否更新为 `gdb/prim-gdb`
  2. 检查 path 是否更新为 `gdb/prim-gdb`
  3. 确认旧配置不再保留
- **预期结果**: `prim-gdb` submodule 配置与新目录结构一致
- **验证方式**: 自动化测试 `test_gitmodules_and_repo_metadata_point_submodule_to_gdb_prim_gdb`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-005: release 脚本按新目录结构打包缓存目录
- **关联 AC**: AC3, AC5
- **类型**: 正向
- **测试数据**: `maze` 启动脚本
- **前置条件**: release 打包逻辑已更新
- **测试步骤**:
  1. 校验 release 目录创建逻辑包含 `release/gdb/maze-db`
  2. 校验 release 目录创建逻辑包含 `release/3rd/misc/cpp-db`
  3. 校验复制命令使用新路径
  4. 校验旧路径复制逻辑已删除
- **预期结果**: release 产物目录结构与仓库新布局一致
- **验证方式**: 自动化测试 `test_release_script_packages_relocated_directories`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-006: 既有测试同步更新新路径断言
- **关联 AC**: AC6
- **类型**: 回归
- **测试数据**: `tests/test_issue_85_maze_py_split_phase1.py`
- **前置条件**: 历史测试已同步调整
- **测试步骤**:
  1. 检查 issue 85 相关测试中的 DB 路径断言
  2. 检查 issue 85 相关测试中的 CDB 路径断言
  3. 确认旧断言已移除
- **预期结果**: 既有测试与新目录结构保持一致
- **验证方式**: 自动化测试 `test_existing_regression_test_is_updated_to_new_db_paths`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-007: GDB portable 路径迁移后仍可被运行时发现
- **关联 AC**: AC7
- **类型**: 错误处理
- **测试数据**: `maze-py/gdb_wrapper.py`
- **前置条件**: `gdb/prim-gdb` 中存在 portable GDB 产物或构建逻辑
- **测试步骤**:
  1. 检查 portable GDB 默认路径是否改为 `gdb/prim-gdb/gdb-portable`
  2. 检查自动初始化与更新命令是否指向 `gdb/prim-gdb`
- **预期结果**: GDB wrapper 不再使用旧根目录 submodule 路径
- **验证方式**: 自动化测试 `test_runtime_and_gdb_scripts_stop_referencing_legacy_locations`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

## Notes for Developer
- 本次新增测试是 TDD 断言，当前仓库状态下预期失败，用于约束迁移实现范围。
- 测试重点覆盖目录存在性、旧路径清理、submodule 配置、release 逻辑与历史测试同步更新。
- 若实现时新增 `3rd/misc/query-cpp-db.py`，建议补充对该脚本新路径的静态断言或在现有测试中扩展检查范围。
