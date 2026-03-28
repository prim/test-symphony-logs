# Test Cases

## 关联计划
Feature: 停用系统 gcore，优先使用 prim-gdb portable 生成 core dump

## Test Case 列表

### TC-001: MAZE_GDB_BIN 优先级覆盖 portable gdb
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: 无，使用单元测试 `prepare/gcmd_test.go`
- **前置条件**: 在临时目录内创建可执行的伪 GDB 文件，以及 `gdb/prim-gdb/gdb-portable`
- **测试步骤**:
  1. 设置 `MAZE_GDB_BIN` 指向临时伪 GDB。
  2. 同时在工作目录下创建 `./gdb/prim-gdb/gdb-portable`。
  3. 调用 `ResolveGDBBin()`。
- **预期结果**: 返回值应为 `MAZE_GDB_BIN` 指向的路径，而不是 portable gdb。
- **验证方式**: Go 单元测试断言返回路径。
- **验收命令**:
  ```bash
  go test ./prepare -run TestResolveGDBBinPrefersEnvOverrideOverPortable
  ```

### TC-002: portable gdb 存在时优先于系统 gdb
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: 无，使用单元测试 `prepare/gcmd_test.go`
- **前置条件**: `MAZE_GDB_BIN` 为空；临时目录下存在 `./gdb/prim-gdb/gdb-portable`
- **测试步骤**:
  1. 清空 `MAZE_GDB_BIN`。
  2. 创建 `./gdb/prim-gdb/gdb-portable`。
  3. 调用 `ResolveGDBBin()`。
- **预期结果**: 返回 `./gdb/prim-gdb/gdb-portable`。
- **验证方式**: Go 单元测试断言解析结果。
- **验收命令**:
  ```bash
  go test ./prepare -run TestResolveGDBBinPrefersPortableBeforeSystem
  ```

### TC-003: corefile.go 禁止重新引入系统 gcore 调用
- **关联 AC**: AC1
- **类型**: 反向
- **测试数据**: 无，使用静态回归测试 `prepare/gcmd_test.go`
- **前置条件**: 仓库包含 `prepare/corefile.go`
- **测试步骤**:
  1. 读取 `prepare/corefile.go` 源码。
  2. 检查是否包含 `exec.Command("gcore"`。
- **预期结果**: 源码中不应包含系统 `gcore` 直接调用字面量。
- **验证方式**: Go 单元测试对源码文本做断言。
- **验收命令**:
  ```bash
  go test ./prepare -run TestCorefileGoDoesNotInvokeSystemGcore
  ```

### TC-004: corefile.go 必须复用 ResolveGDBBin
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: 无，使用静态回归测试 `prepare/gcmd_test.go`
- **前置条件**: 仓库包含 `prepare/corefile.go`
- **测试步骤**:
  1. 读取 `prepare/corefile.go` 源码。
  2. 检查是否调用 `ResolveGDBBin()`。
- **预期结果**: 源码中必须出现 `ResolveGDBBin()`，保证 core dump 与 GDB Server 共用解析入口。
- **验证方式**: Go 单元测试对源码文本做断言。
- **验收命令**:
  ```bash
  go test ./prepare -run TestCorefileGoUsesResolveGDBBin
  ```

### TC-005: corefile.go 必须构造 GDB generate-core-file/gcore 命令
- **关联 AC**: AC2
- **类型**: 正向
- **测试数据**: 无，使用静态回归测试 `prepare/gcmd_test.go`
- **前置条件**: 仓库包含 `prepare/corefile.go`
- **测试步骤**:
  1. 读取 `prepare/corefile.go` 源码。
  2. 检查是否出现 `generate-core-file` 或通过 GDB `-ex` 执行 `gcore` 的指令片段。
- **预期结果**: 源码体现通过 resolved GDB 执行抓 core，而不是外部系统命令。
- **验证方式**: Go 单元测试对源码文本做断言。
- **验收命令**:
  ```bash
  go test ./prepare -run TestCorefileGoBuildsGenerateCoreCommand
  ```

### TC-006: prepare 构建与主链路回归
- **关联 AC**: AC5, AC6
- **类型**: 烟雾
- **测试数据**: `testdata/python/20260129-complex-types-311`
- **前置条件**: 功能实现完成并可成功编译；testdata 子模块已初始化
- **测试步骤**:
  1. 执行 `./maze --build`。
  2. 运行一个 prepare 主链路 testdata 用例。
- **预期结果**: 编译成功，且至少一个 prepare 主链路用例通过。
- **验证方式**: 命令行构建与 testdata 冒烟测试。
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

## 测试文件

- `prepare/gcmd_test.go`: 覆盖 GDB 二进制优先级、禁止系统 gcore 回归、要求 corefile 复用 `ResolveGDBBin()`、要求改为 GDB 指令抓 core。

## 说明

- 这些测试遵循 TDD 目的：在功能尚未实现前，新增回归测试应失败。
- 本轮仅新增测试，不修改功能实现代码。
- 当前仓库环境执行 `go test ./prepare/...` 时受 `grocksdb` 头文件缺失影响，属于外部构建依赖问题；新增测试仍已按 issue 要求落到 `prepare/gcmd_test.go`，并会在相关实现完成后对目标行为形成约束。
