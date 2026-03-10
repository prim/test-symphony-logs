# Test Cases

## 关联计划
Feature: 将 gdbcommand.py 迁移到 gdb-scripts 目录

## Test File(s)
- `tests/test_issue_70_gdbcommand_migration.py`: 通过仓库结构检查和轻量级模块导入验证本次迁移的关键验收点

## Test Case 列表

### TC-001: gdbcommand 模块迁移到 gdb-scripts
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 仓库源码树
- **前置条件**: 在项目根目录执行测试
- **测试步骤**:
  1. 检查根目录是否仍存在 `gdbcommand.py`
  2. 检查 `gdb-scripts/gdbcommand.py` 是否存在
- **预期结果**: 根目录文件不存在，`gdb-scripts/gdbcommand.py` 存在
- **验证方式**: 自动化单元测试 `test_gdbcommand_module_is_moved_under_gdb_scripts`

### TC-002: .gdbcommand.py 导入新位置的 gdbcommand 模块
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: `.gdbcommand.py`
- **前置条件**: 仓库包含迁移后的导入语句
- **测试步骤**:
  1. 读取 `.gdbcommand.py`
  2. 检查是否包含 `import gdbcommand`
  3. 检查是否包含 `from gdbcommand import *`
- **预期结果**: 入口脚本显式导入迁移后的模块，并将其导出的类/函数暴露到全局命名空间
- **验证方式**: 自动化单元测试 `test_gdbcommand_entry_imports_migrated_module`

### TC-003: release 打包逻辑不再复制根目录 gdbcommand.py
- **关联 AC**: AC1, AC2
- **类型**: 反向
- **测试数据**: `maze`
- **前置条件**: 仓库包含最新 release 逻辑
- **测试步骤**:
  1. 读取 `maze`
  2. 检查是否仍有 `cp gdbcommand.py release`
  3. 检查是否保留 `cp gdb-scripts/*.py release/gdb-scripts`
- **预期结果**: 不再从根目录复制 `gdbcommand.py`，而是统一从 `gdb-scripts/` 打包 GDB 脚本
- **验证方式**: 自动化单元测试 `test_release_script_copies_gdbcommand_only_from_gdb_scripts`

### TC-004: 迁移后的 gdbcommand 模块仍注册核心 GDB 命令
- **关联 AC**: AC4
- **类型**: 兼容性
- **测试数据**: `gdb-scripts/gdbcommand.py`
- **前置条件**: 测试通过 stub gdb 环境导入模块
- **测试步骤**:
  1. 使用 stub `gdb` 模块导入 `gdb-scripts/gdbcommand.py`
  2. 记录注册到 `gdb.Command` 的命令名
  3. 校验 `offsets-of`、`offsets-of-value`、`tcmalloc`、`jemalloc`
- **预期结果**: 核心命令在迁移后仍然被注册
- **验证方式**: 自动化单元测试 `test_migrated_gdbcommand_registers_core_commands`

## Cases
1. `test_gdbcommand_module_is_moved_under_gdb_scripts`: 验证文件位置已迁移
2. `test_gdbcommand_entry_imports_migrated_module`: 验证入口脚本导入新模块
3. `test_release_script_copies_gdbcommand_only_from_gdb_scripts`: 验证 release 打包逻辑无旧路径残留
4. `test_migrated_gdbcommand_registers_core_commands`: 验证 GDB 命令注册兼容性

## Notes for Developer
- 当前仓库中的 `testdata/` 子模块未展开，因此本次先提供针对迁移行为的可执行单元测试；AC5 仍应在具备完整 `testdata` 环境后使用 `python3 testdata/run_test.py cpp/20260304-stack-multithread` 进行回归验证。
- 这些测试按 TDD 预期应在功能未实现时失败：当前根目录仍存在 `gdbcommand.py`，`gdb-scripts/` 下缺少迁移后的模块，`.gdbcommand.py` 也尚未显式导入该模块。
