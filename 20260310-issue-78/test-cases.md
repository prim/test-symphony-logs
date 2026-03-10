# Test Cases

## 关联计划
Feature: GDB 目录重组

## Test Case 列表

### TC-001: GDB 目录重组后的目标结构存在
- **关联 AC**: AC1, AC2
- **类型**: 正向
- **测试数据**: 仓库工作区目录结构
- **前置条件**: 开发分支已完成目录迁移
- **测试步骤**:
  1. 检查仓库根目录下是否存在 `gdb/`
  2. 检查 `gdb/` 下是否包含 `gdb-data/`、`gdb-scripts/`、`gdb-server/`
  3. 检查仓库根目录下的 `gdb-data/`、`gdb-scripts/`、`gdb_server/` 是否已移除
- **预期结果**: 新目录结构存在，旧目录不再存在于仓库根目录
- **验证方式**: 自动化单元测试 `test_target_gdb_directory_layout_is_present`
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_78_gdb_directory_reorganization
  ```

### TC-002: 代码中的 GDB 路径引用全部更新为新目录
- **关联 AC**: AC3, AC4, AC5
- **类型**: 正向
- **测试数据**: `.maze.py`、`.gdbcommand.py`、`maze`、`gdb/gdb-scripts/`、`gdb/gdb-server/`
- **前置条件**: 目录迁移与路径替换已完成
- **测试步骤**:
  1. 检查 `.maze.py` 是否使用 `gdb/gdb-scripts` 和 `gdb/gdb-data`
  2. 检查 `.gdbcommand.py` 是否使用 `gdb/gdb-scripts` 和 `gdb/gdb-data`
  3. 检查 `maze` 发布/更新逻辑是否改为 `gdb/` 下的新路径
  4. 扫描 `gdb/gdb-scripts/` 与 `gdb/gdb-server/` 中的代码，确认不存在旧路径引用
- **预期结果**: 所有代码只引用新路径，不再出现 `gdb-data/`、`gdb-scripts/`、`gdb_server/` 的旧根目录引用
- **验证方式**: 自动化单元测试 `test_core_entrypoints_reference_new_gdb_paths`、`test_no_legacy_gdb_paths_remain_in_runtime_code`
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_78_gdb_directory_reorganization
  ```

### TC-003: .gitignore 忽略规则跟随新目录结构更新
- **关联 AC**: AC6
- **类型**: 正向
- **测试数据**: `.gitignore`
- **前置条件**: `.gitignore` 已更新
- **测试步骤**:
  1. 检查 `.gitignore` 是否包含 `gdb/gdb-data/*`
  2. 检查 `.gitignore` 是否移除了旧的 `gdb-data/*`
- **预期结果**: 临时文件忽略规则指向 `gdb/gdb-data/`
- **验证方式**: 自动化单元测试 `test_gitignore_tracks_new_gdb_data_location`
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_78_gdb_directory_reorganization
  ```

### TC-004: 发布脚本随 GDB 目录重组同步调整
- **关联 AC**: AC4, AC5
- **类型**: 边界
- **测试数据**: `maze`
- **前置条件**: 发布逻辑已同步迁移路径
- **测试步骤**:
  1. 检查 `maze` 中 release 目录初始化逻辑
  2. 检查 `maze` 中脚本复制逻辑
  3. 确认发布过程不再依赖旧目录名
- **预期结果**: release 使用 `release/gdb/gdb-data`、`release/gdb/gdb-scripts`，并从 `gdb/` 下复制资源
- **验证方式**: 自动化单元测试 `test_release_script_uses_reorganized_gdb_paths`
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_78_gdb_directory_reorganization
  ```

## Notes for Developer

- 这些测试是 TDD 约束，当前应失败，直到目录迁移和路径修复全部完成。
- 建议优先处理 `.maze.py`、`.gdbcommand.py`、`maze` 三个入口文件，再统一替换 `gdb/gdb-scripts/` 与 `gdb/gdb-server/` 下的旧路径引用。
- 如果保留兼容层，请避免在运行时代码中继续依赖仓库根目录旧路径，否则测试仍会失败。
