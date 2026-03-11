# Test Cases

## 关联计划
Feature: 将 `.maze.py` 全部重写为 Go `prepare` 包

## Test File(s)

- `qa/issue110/prepare_migration_test.go`: 仓库级迁移测试，验证 prepare 包脚手架、入口集成、旧 JSON 初始化清理、遗留 Python 预处理移除与 GDB 脚本保留

## Test Case 列表

### TC-001: prepare 包基础脚手架存在
- **关联 AC**: AC1, AC5, AC6
- **类型**: 正向
- **测试数据**: 无需新建，检查源码结构
- **前置条件**: 仓库包含迁移后的 Go 源码
- **测试步骤**:
  1. 执行 `go test ./qa/issue110 -run TestPreparePackageScaffoldExists`
  2. 检查 `prepare/` 目录下关键文件是否存在
- **预期结果**: `prepare/prepare.go`、`context.go`、`args.go`、`cmd.go`、`gdb_client.go`、`gdb_server.go`、`process.go`、`symbols.go`、`fill.go` 等核心文件全部存在
- **验证方式**: 自动化 Go 测试

### TC-002: main.go 集成 prepare.Run
- **关联 AC**: AC1, AC2, AC7, AC8
- **类型**: 正向
- **测试数据**: 无需新建，检查源码内容
- **前置条件**: 迁移后的入口逻辑已落地
- **测试步骤**:
  1. 执行 `go test ./qa/issue110 -run TestMainUsesPrepareRun`
  2. 检查 `main.go` 是否导入 `maze/prepare`
  3. 检查是否调用 `prepare.Run()`
- **预期结果**: `main.go` 在主流程中明确集成 `prepare.Run()`
- **验证方式**: 自动化 Go 测试

### TC-003: common 初始化不再依赖 .storage.json / .cpp.json
- **关联 AC**: AC1, AC9
- **类型**: 反向
- **测试数据**: 无需新建，检查源码内容
- **前置条件**: prepare 阶段已改为直接填充 `common.S` / `common.Cpp`
- **测试步骤**:
  1. 执行 `go test ./qa/issue110 -run TestCommonInitNoLongerDependsOnLegacyJSONBootstrap`
  2. 检查 `common/common.go` 不再调用 `initConfig()`
  3. 检查 `common/var.go` 不再读取 `.storage.json` / `.cpp.json`
- **预期结果**: 初始化流程完全摆脱旧 JSON 中间文件
- **验证方式**: 自动化 Go 测试

### TC-004: 遗留 Python 预处理被删除且 GDB 脚本保留
- **关联 AC**: AC9, AC10
- **类型**: 反向/兼容性
- **测试数据**: 无需新建，检查仓库文件
- **前置条件**: 迁移收尾已完成
- **测试步骤**:
  1. 执行 `go test ./qa/issue110 -run TestLegacyPythonPreprocessIsRemovedButGdbScriptsRemain`
  2. 检查 `.maze.py` 与 `maze-py/` 不存在
  3. 检查 `gdb/gdb-scripts/*.py` 与 `gdb/gdb-server/gdb_socket_server.py` 仍存在
- **预期结果**: Python 预处理层被移除，但 GDB 内部 Python 脚本被保留
- **验证方式**: 自动化 Go 测试

### TC-005: maze 入口不再是 Python wrapper
- **关联 AC**: AC2, AC8, AC9
- **类型**: 反向
- **测试数据**: 无需新建，检查入口文件内容
- **前置条件**: 统一入口已迁移为 Go 二进制或等价 wrapper
- **测试步骤**:
  1. 执行 `go test ./qa/issue110 -run TestMazeEntryIsNoLongerPythonWrapper`
  2. 检查 `maze` 是否仍以 Python shebang 开头
  3. 检查是否仍包含 `do_build`、`do_tar`、`do_release` 等旧逻辑
- **预期结果**: 入口不再依赖 Python wrapper 逻辑
- **验证方式**: 自动化 Go 测试

### TC-006: GDB 脚本仍基于 GDB Python API
- **关联 AC**: AC10
- **类型**: 兼容性
- **测试数据**: 无需新建，检查脚本内容
- **前置条件**: GDB 脚本仍由 GDB 内部 Python 执行
- **测试步骤**:
  1. 执行 `go test ./qa/issue110 -run TestGdbScriptsStillUsePythonAPI`
  2. 检查 `gdb_socket_server.py` 是否继续 `import gdb`
- **预期结果**: GDB 脚本保持 Python API 实现，不被误删或误迁移
- **验证方式**: 自动化 Go 测试

### TC-007: Python 主流程回归
- **关联 AC**: AC1, AC3, AC6, AC8
- **类型**: 正向/准确性
- **测试数据**: `testdata/python/20260129-complex-types-311`
- **前置条件**: 迁移后 `run_test.py` 仍可调用 Maze 完成 `--tar` 分析
- **测试步骤**:
  1. 执行下列验收命令
  2. 观察 Python 3.11 复杂类型统计是否满足 validate.py 断言
- **预期结果**: 用例通过，说明 Go prepare 可以替代原 Python 预处理完成 Python 语言检测、符号与类型信息准备
- **验证方式**: 自动化测试命令
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-008: C++ 多线程栈变量回归
- **关联 AC**: AC1, AC4, AC5, AC6, AC7
- **类型**: 正向/边界
- **测试数据**: `testdata/cpp/20260304-stack-multithread`
- **前置条件**: 迁移后 C++ DWARF 与多线程栈局部变量收集仍可用
- **测试步骤**:
  1. 执行下列验收命令
  2. 观察 validate.py 是否识别 `ThreadObject` 或至少分析完成无崩溃
- **预期结果**: 用例通过，说明多线程 C++ 栈变量分析未因迁移退化
- **验证方式**: 自动化测试命令
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260304-stack-multithread
  ```

### TC-009: `--tar` 单入口分析能力回归
- **关联 AC**: AC2, AC8
- **类型**: 正向
- **测试数据**: 可复用 `python/20260129-complex-types-311` 或 `cpp/20260304-stack-multithread`
- **前置条件**: Go 统一入口可接受 `--tar` 参数
- **测试步骤**:
  1. 通过 `run_test.py` 间接执行 `maze --tar ... --text --json-output`
  2. 确认 `maze-result.json` 生成且 validate 通过
- **预期结果**: `--tar` 一键分析继续可用
- **验证方式**: 自动化回归测试

### TC-010: 模式兼容与遗留依赖清理检查
- **关联 AC**: AC7, AC9, AC10
- **类型**: 兼容性/边界
- **测试数据**: 无需新建，结合源码检查与回归执行
- **前置条件**: standalone/postmanprofile 分发逻辑已迁移
- **测试步骤**:
  1. 检查 `main.go` 参数与模式入口设计
  2. 检查遗留 `.maze.py`/`maze-py` 是否删除
  3. 保留 `gdb-scripts/*.py`
- **预期结果**: 模式迁移与文件清理一致，不破坏 GDB 内部脚本兼容性
- **验证方式**: 自动化源码测试 + 回归用例

## Notes for Developer

- 本次新增的是 **TDD 迁移守护测试**，当前仓库状态下预期失败，用于明确迁移完成标准。
- 测试重点不是功能修复，而是约束迁移后的架构结果：必须有 `prepare/` 包、`main.go` 必须调用 `prepare.Run()`、`common` 不得再从 `.storage.json` / `.cpp.json` 初始化。
- `gdb-scripts/*.py` 与 `gdb/gdb-server/gdb_socket_server.py` 必须继续保留；这些脚本是 GDB 内部 Python API 的一部分，不在迁移删除范围内。
- 验收时仍需执行现有 `testdata` 用例，建议至少覆盖：
  - `python/20260129-complex-types-311`
  - `cpp/20260304-stack-multithread`
