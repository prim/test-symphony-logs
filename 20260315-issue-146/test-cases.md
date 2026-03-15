# Test Cases

## 关联计划
Feature: 删除多余的 `--run` 选项

## Test Case 列表

### TC-001: CLI 不再注册 `--run` 选项
- **关联 AC**: AC1 `main.go` 删除 `--run` flag 定义与帮助文案
- **类型**: 反向
- **测试数据**: 新增 `qa/issue146/remove_run_option_test.go`
- **前置条件**: 仓库包含最新 issue #146 开发改动
- **测试步骤**:
  1. 解析 `main.go` 的 `parseArgs()` 实现。
  2. 检查是否仍存在 `flag.BoolVar(&RunAfterBuild, "run", ...)`。
  3. 检查 `main.go` 是否仍包含 `RunAfterBuild` 或 `run immediately after build`。
- **预期结果**: `main.go` 不再注册 `--run`，也不再引用 `RunAfterBuild`。
- **验证方式**: Go AST + 源码文本断言
- **验收命令**:
  ```bash
  go test ./qa/issue146
  ```

### TC-002: `--build` 后退出逻辑仅依赖分析意图
- **关联 AC**: AC2 `if !hasAnalysisIntent() { os.Exit(0) }`
- **类型**: 正向
- **测试数据**: 新增 `qa/issue146/remove_run_option_test.go`
- **前置条件**: `handleCommandMode()` 已完成 issue #146 实现
- **测试步骤**:
  1. 解析 `main.go` 中 `handleCommandMode()`。
  2. 找到 `if BuildMode { ... }` 分支内的退出条件。
  3. 校验条件文本是否为 `!hasAnalysisIntent()`，且不再包含 `RunAfterBuild`。
- **预期结果**: 构建后的退出逻辑不再依赖 `--run`。
- **验证方式**: Go AST 条件表达式断言
- **验收命令**:
  ```bash
  go test ./qa/issue146
  ```

### TC-003: 公共变量定义移除 `RunAfterBuild`
- **关联 AC**: AC3 `common/var.go` 删除 `RunAfterBuild`
- **类型**: 正向
- **测试数据**: 新增 `qa/issue146/remove_run_option_test.go`
- **前置条件**: `common/var.go` 已同步清理
- **测试步骤**:
  1. 读取 `common/var.go`。
  2. 搜索 `RunAfterBuild` 标识符。
- **预期结果**: `common/var.go` 中不存在 `RunAfterBuild`。
- **验证方式**: 文本断言
- **验收命令**:
  ```bash
  go test ./qa/issue146
  ```

### TC-004: loop 主命令不再拼接 `--run`
- **关联 AC**: AC4 清理命令构造中的 `--run` 引用
- **类型**: 正向
- **测试数据**: 新增 `qa/issue146/remove_run_option_test.go`
- **前置条件**: `doLoop()` 已按新 CLI 更新
- **测试步骤**:
  1. 读取 `main.go`。
  2. 检查 `doLoop()` 默认命令切片。
  3. 确认旧命令 `./maze --build --run --mode postman --tag dev` 不再存在。
  4. 确认新命令为 `./maze --build --mode postman --tag dev`。
- **预期结果**: loop 模式默认命令与新 CLI 保持一致。
- **验证方式**: 文本断言
- **验收命令**:
  ```bash
  go test ./qa/issue146
  ```

### TC-005: 活跃文档与测试脚本不再示例 `--run`
- **关联 AC**: AC5 清理文档和测试脚本中的 `--run` 引用
- **类型**: 兼容性
- **测试数据**: 新增 `qa/issue146/remove_run_option_test.go`
- **前置条件**: 开发者已同步更新用户文档与当前仍活跃的脚本
- **测试步骤**:
  1. 读取 `doc/build_and_run.md`。
  2. 读取 `tests/test_maze_loop_integration.py`。
  3. 读取 `cmd/postman.py` 与 `AGENTS.md`。
  4. 检查这些文件中是否仍包含 `--run` 的示例命令。
- **预期结果**: 活跃文档和脚本不再依赖 `--run` 用法。
- **验证方式**: 文本断言
- **验收命令**:
  ```bash
  go test ./qa/issue146
  ```

## 额外覆盖说明

- **Happy path**: 构建后只要存在 `--pid` / `--tar` / standalone 分析意图，就应继续执行，无需 `--run`。
- **边界条件**: 仅执行 `./maze --build` 时仍应直接退出，不触发分析流程。
- **错误处理**: 若源码仍残留 `RunAfterBuild`、flag 注册、loop 旧命令或文档旧示例，测试立即失败并给出精确文件位置。

## 本轮测试代码

- `qa/issue146/remove_run_option_test.go`: 用 AST + 文本断言约束 CLI、变量、循环命令和活跃文档清理范围。

## 当前结果

- 已执行 `go test ./qa/issue146`
- 当前按预期失败，说明功能尚未实现完成，符合 TDD 阶段目标

## Notes for Developer

- 测试把“删除 `--run`”视为一个范围清理任务，不仅验证 `main.go` 和 `common/var.go`，也验证当前仍活跃的脚本与文档。
- 若决定保留某些历史文档中的旧命令，需要同步调整测试范围；否则请统一清理，避免用户继续复制过时命令。
