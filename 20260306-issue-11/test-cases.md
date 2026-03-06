# Test Cases

## 关联计划
Feature: `./maze --byebye` 输出 `byebye`

## Test File(s)
- `tests/__init__.py`: 使 `python3 -m unittest tests.test_maze_cli` 可导入 `tests` 包
- `tests/test_maze_cli.py`: CLI 行为回归测试，覆盖 `--byebye` 的输出、退出码和短路行为

## Test Case 列表

### TC-001: `--byebye` 输出固定文本并正常退出
- **关联 AC**: AC1, AC2
- **类型**: 正向
- **测试数据**: 无，直接执行 CLI
- **前置条件**: 仓库根目录可执行 `python3 maze`
- **测试步骤**:
  1. 执行 `python3 -m unittest tests.test_maze_cli.MazeByeByeCliTest.test_byebye_prints_expected_output_and_exits_zero`
  2. 观察进程退出码、stdout、stderr
- **预期结果**:
  - 退出码为 0
  - stdout 严格等于 `byebye\n`
  - stderr 为空
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_maze_cli.MazeByeByeCliTest.test_byebye_prints_expected_output_and_exits_zero
  ```

### TC-002: `--byebye` 优先于其他运行参数生效
- **关联 AC**: AC3
- **类型**: 边界
- **测试数据**: 无，直接执行 CLI
- **前置条件**: 仓库根目录可执行 `python3 maze`
- **测试步骤**:
  1. 执行 `python3 -m unittest tests.test_maze_cli.MazeByeByeCliTest.test_byebye_short_circuits_other_runtime_requirements`
  2. 观察传入无效 `--pid` 时程序是否仍直接返回 `byebye`
- **预期结果**:
  - 退出码为 0
  - stdout 严格等于 `byebye\n`
  - stderr 为空
  - 不进入常规分析路径，不依赖有效 PID
- **验证方式**: 自动化单元测试
- **验收命令**:
  ```bash
  python3 -m unittest tests.test_maze_cli.MazeByeByeCliTest.test_byebye_short_circuits_other_runtime_requirements
  ```

## Notes for Developer
- `--byebye` 适合作为最早处理的 CLI 快捷参数，避免落入默认帮助输出或分析流程。
- 测试要求 stdout 精确匹配 `byebye\n`，避免混入帮助文本、日志或额外前缀。
