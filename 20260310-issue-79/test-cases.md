# Test Cases

## 关联计划
Feature: 将 .postman_lz4.py 和 .postman_prelude.py 移动到 cmd/ 目录

## Test Case 列表

### TC-001: postman 脚本迁移到 cmd 目录
- **关联 AC**: AC1, AC5
- **类型**: 正向
- **测试数据**: 仓库工作树文件
- **前置条件**: 功能分支包含脚本迁移提交
- **测试步骤**:
  1. 检查仓库根目录不存在 `.postman_lz4.py` 和 `.postman_prelude.py`
  2. 检查 `cmd/postman_lz4.py` 和 `cmd/postman_prelude.py` 存在
- **预期结果**: 两个脚本仅存在于 `cmd/` 目录，根目录旧文件已删除
- **验证方式**: 自动化测试 `tests/test_issue_79_move_postman_scripts.py::test_postman_helper_scripts_are_moved_into_cmd_directory`

### TC-002: maze 脚本引用全部更新为 cmd 路径
- **关联 AC**: AC2, AC6
- **类型**: 正向
- **测试数据**: 仓库工作树文件
- **前置条件**: 功能分支包含 maze 脚本路径更新
- **测试步骤**:
  1. 读取 `maze` 文件内容
  2. 验证下载更新、release 打包、tar 分析流程中旧根目录路径已移除
  3. 验证对应位置改为 `cmd/` 路径
- **预期结果**: `maze` 中所有 `.postman_*` 引用都使用新的 `cmd/` 路径
- **验证方式**: 自动化测试 `tests/test_issue_79_move_postman_scripts.py::test_maze_updates_all_postman_script_references_to_cmd_directory`

### TC-003: cmd/postman.py 调用新的 prelude 路径
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: 仓库工作树文件
- **前置条件**: 功能分支包含 `cmd/postman.py` 路径更新
- **测试步骤**:
  1. 读取 `cmd/postman.py`
  2. 验证旧命令 `python .postman_prelude.py` 不存在
  3. 验证新命令 `python cmd/postman_prelude.py` 存在
- **预期结果**: `cmd/postman.py` 仅使用迁移后的 prelude 路径
- **验证方式**: 自动化测试 `tests/test_issue_79_move_postman_scripts.py::test_cmd_postman_uses_cmd_prefixed_prelude_path`

### TC-004: postman-main.go 调用新的 prelude 路径
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: 仓库工作树文件
- **前置条件**: 功能分支包含 `postman/postman-main.go` 路径更新
- **测试步骤**:
  1. 读取 `postman/postman-main.go`
  2. 验证旧调用 `exec.Command("python", ".postman_prelude.py", ...)` 不存在
  3. 验证新调用 `exec.Command("python", "cmd/.postman_prelude.py", ...)` 存在
- **预期结果**: Go 侧执行 prelude 时使用新的 `cmd/` 路径
- **验证方式**: 自动化测试 `tests/test_issue_79_move_postman_scripts.py::test_postman_main_go_uses_cmd_prefixed_prelude_path`

## Notes for Developer

- 这些测试是静态回归测试，覆盖文件移动与路径引用更新，不涉及功能实现逻辑修改。
- 当前仓库状态下新测试应失败，这符合 TDD 预期，可用于驱动后续实现。
