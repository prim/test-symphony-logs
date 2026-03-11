# Test Cases

## 关联计划
Feature: 将 maze-db 和 prim-gdb 目录迁移至 gdb/ 子目录

## Test Case 列表

### TC-001: 目录迁移后的 GDB 根目录布局正确
- **关联 AC**: AC1, AC2, AC3
- **类型**: 正向
- **测试数据**: 仓库工作区目录结构
- **前置条件**: 功能分支已完成目录迁移
- **测试步骤**:
  1. 检查 `gdb/` 目录是否存在。
  2. 检查 `gdb/maze-db/`、`gdb/prim-gdb/`、`gdb/gdb-scripts/`、`gdb/gdb-server/` 是否存在。
  3. 检查仓库根目录下是否还存在 `maze-db/` 和 `prim-gdb/`。
- **预期结果**: 新目录存在，旧根目录不存在。
- **验证方式**: 自动化单元测试 `tests/test_issue_101_move_maze_db_and_prim_gdb.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-002: .gitmodules 中 prim-gdb 子模块路径已更新
- **关联 AC**: AC2, AC4, AC8
- **类型**: 正向
- **测试数据**: `.gitmodules`
- **前置条件**: 完成子模块路径迁移
- **测试步骤**:
  1. 读取 `.gitmodules`。
  2. 验证 `prim-gdb` 的 `path` 为 `gdb/prim-gdb`。
  3. 验证不再保留旧值 `path = prim-gdb`。
- **预期结果**: 子模块路径指向新目录。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-003: maze_db.py 使用新的 ctypes 缓存目录
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: `maze_db.py`
- **前置条件**: 已完成路径替换
- **测试步骤**:
  1. 读取 `maze_db.py`。
  2. 验证 `DB = DBC("gdb/maze-db")` 存在。
  3. 验证旧路径 `DB = DBC("maze-db")` 不存在。
- **预期结果**: 运行时 DB 初始化仅使用新路径。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-004: gdb_wrapper 使用新的 portable GDB 路径和子模块命令
- **关联 AC**: AC6, AC8
- **类型**: 正向
- **测试数据**: `maze-py/gdb_wrapper.py`
- **前置条件**: 已完成 GDB 路径迁移
- **测试步骤**:
  1. 检查 portable GDB 路径是否为 `./gdb/prim-gdb/gdb-portable`。
  2. 检查 `git submodule init/update` 是否使用 `gdb/prim-gdb`。
  3. 检查构建命令中的 `cd` 是否切换到 `gdb/prim-gdb`。
  4. 验证旧路径命令和旧路径字面量不存在。
- **预期结果**: GDB 包装器完全切换到新路径。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-005: 发布脚本使用新的 release/gdb/maze-db 路径
- **关联 AC**: AC7
- **类型**: 正向
- **测试数据**: `maze`
- **前置条件**: 已完成 release 脚本更新
- **测试步骤**:
  1. 检查 release 目录创建逻辑。
  2. 验证创建的是 `release/gdb/maze-db`。
  3. 验证复制来源是 `gdb/maze-db/*`。
  4. 验证旧路径 `release/maze-db` 不再出现。
- **预期结果**: release 输出目录结构与迁移后目录一致。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-006: cleanup 命令清理新的缓存目录
- **关联 AC**: AC7
- **类型**: 边界
- **测试数据**: `maze`
- **前置条件**: 已完成 cleanup 路径更新
- **测试步骤**:
  1. 检查 `--cleanup` 参数帮助文本。
  2. 检查实际删除命令是否为 `rm gdb/maze-db/ctypes-*`。
  3. 验证旧路径 `rm maze-db/ctypes-*` 不再存在。
- **预期结果**: 清理逻辑不会误删旧根目录路径。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-007: 运行时代码中不再残留旧路径引用
- **关联 AC**: AC5, AC6, AC7
- **类型**: 反向
- **测试数据**: `maze_db.py`、`maze`、`maze-py/gdb_wrapper.py`
- **前置条件**: 完成所有路径迁移
- **测试步骤**:
  1. 扫描上述运行时代码文件。
  2. 检查是否残留 `maze-db`、`./prim-gdb/gdb-portable`、`git submodule init prim-gdb`、`release/maze-db` 等旧字面量。
- **预期结果**: 不存在旧路径引用。
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-008: 回归验证 Python 复杂类型测试无回归
- **关联 AC**: AC9, AC10
- **类型**: 回归
- **测试数据**: `testdata/python/20260129-complex-types-311`
- **前置条件**: `testdata` 子模块已初始化，项目已完成实现
- **测试步骤**:
  1. 执行 `./maze --build`。
  2. 执行 `python3 testdata/run_test.py python/20260129-complex-types-311`。
  3. 检查 `maze.log`、`maze.py.log` 无新增异常路径错误。
- **预期结果**: 构建成功，测试通过，日志中无旧路径导致的报错。
- **验证方式**: 自动化测试 + 日志检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

## 测试文件
- `tests/test_issue_101_move_maze_db_and_prim_gdb.py`: 覆盖目录迁移、子模块路径、运行时路径与发布脚本路径更新。

## Notes for Developer
- 本次测试采用静态结构断言，目标是确保迁移后不会遗漏任何硬编码路径。
- 新测试在当前实现下应失败，这是符合 TDD 预期的。
- 验收阶段仍需实际执行 `./maze --build` 和 `python3 testdata/run_test.py python/20260129-complex-types-311`，验证运行时行为与回归情况。
