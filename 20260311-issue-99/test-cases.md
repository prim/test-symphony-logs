# Test Cases

## 关联计划
Feature: 将 dd 包迁移至 common/dd 目录

## 测试文件
- `tests/test_issue_99_move_dd_to_common.py`：验证 dd 包迁移后的目录布局、导入路径替换，以及仓库中不再残留旧导入。

## Test Case 列表

### TC-001: dd 包文件迁移到 common/dd
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 仓库源码树
- **前置条件**: 开发者已完成目录迁移
- **测试步骤**:
  1. 检查 `common/dd/` 目录是否存在。
  2. 检查 `common/dd/` 中是否包含 `vm_map.go`、`vm_tcmalloc_search_pagemap.go`、`vm_zero.go`。
- **预期结果**: `common/dd/` 存在，且包含原 `dd/` 下全部 Go 文件。
- **验证方式**: 自动化测试 `tests/test_issue_99_move_dd_to_common.py::test_common_dd_contains_all_migrated_go_files`
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_99_move_dd_to_common
  ```

### TC-002: 根目录 dd 目录已删除
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: 仓库源码树
- **前置条件**: 开发者已完成目录迁移
- **测试步骤**:
  1. 检查根目录下是否仍存在 `dd/`。
- **预期结果**: 根目录下不存在 `dd/` 目录。
- **验证方式**: 自动化测试 `tests/test_issue_99_move_dd_to_common.py::test_legacy_dd_directory_is_removed`
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_99_move_dd_to_common
  ```

### TC-003: main.go 与 http.go 使用新导入路径
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: `main.go`、`http.go`
- **前置条件**: 开发者已修改 import 语句
- **测试步骤**:
  1. 检查 `main.go` 是否包含 `. "maze/common/dd"`。
  2. 检查 `http.go` 是否包含 `. "maze/common/dd"`。
  3. 确认两个文件中都不再出现 `. "maze/dd"`。
- **预期结果**: 两个文件均仅使用 `maze/common/dd` 导入路径。
- **验证方式**: 自动化测试 `tests/test_issue_99_move_dd_to_common.py::test_main_and_http_import_common_dd`
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_99_move_dd_to_common
  ```

### TC-004: 仓库内无旧 maze/dd 导入残留
- **关联 AC**: AC3
- **类型**: 边界
- **测试数据**: 全部 `*.go` 文件
- **前置条件**: 开发者已完成迁移
- **测试步骤**:
  1. 扫描仓库中的所有 Go 文件。
  2. 检查是否仍存在 `"maze/dd"` 字面量。
- **预期结果**: 无任何 Go 文件残留旧导入路径。
- **验证方式**: 自动化测试 `tests/test_issue_99_move_dd_to_common.py::test_no_go_file_references_legacy_maze_dd_import`
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_issue_99_move_dd_to_common
  ```

### TC-005: 项目编译通过
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: 项目源码
- **前置条件**: 完成 dd 包迁移与导入更新
- **测试步骤**:
  1. 在项目根目录执行编译命令。
- **预期结果**: `./maze --build` 执行成功，无编译错误。
- **验证方式**: 手动/CI 执行编译命令并检查退出码与输出。
- **验收命令**:
  ```bash
  ./maze --build
  ```

### TC-006: Python 复杂类型回归验证
- **关联 AC**: AC5
- **类型**: 回归
- **测试数据**: `testdata/python/20260129-complex-types-311`
- **前置条件**: `testdata` 子模块已初始化，编译完成
- **测试步骤**:
  1. 执行指定 Python 回归测试用例。
- **预期结果**: 测试通过，无功能回归。
- **验证方式**: 运行统一测试入口 `run_test.py`。
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

## 说明
- 本次新增自动化测试聚焦目录迁移与导入路径替换，能够在功能实现前稳定失败，符合先写测试的 TDD 目标。
- AC4 与 AC5 依赖真实构建和测试环境，保留为后续实现完成后的验收命令。
- 该需求主要是代码组织迁移，不涉及新的内存对象统计逻辑，因此无需新增 `testdata/` 场景数据。

## Notes for Developer
- 迁移 `dd/` 时请保持包名 `package dd` 不变，仅调整目录位置与 import 路径。
- 完成迁移后需删除根目录遗留的 `dd/` 目录，否则 TC-002 会失败。
- 除 `main.go` 与 `http.go` 外，若新增其他 Go 文件引用了 `maze/dd`，TC-004 也会直接失败。
