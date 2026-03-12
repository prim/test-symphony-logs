# Test Cases

## 关联计划
Feature: 修复 standalone 模式下 HttpSetup 缺失 base URL 导致 panic

## Test File(s)
- `tests/test_issue_126_standalone_http_setup_base_url.py`：为 HTTP 初始化职责拆分、base URL 校验、降级路径与构建回归提供 TDD 回归测试

## Test Case 列表

### TC-001: standalone 未配置远端地址时跳过 `/new` 注册
- **关联 AC**: AC1, AC3
- **类型**: 正向 / 边界
- **测试数据**: 无额外 testdata，源码级回归测试
- **前置条件**: 仓库可读取 `http.go`
- **测试步骤**:
  1. 检查 `HttpSetup()` 是否仍无条件调用 `HttpJsonRequest("new", ...)`
  2. 检查是否引入“跳过 `/new`”的显式分支或日志语义
- **预期结果**: standalone 本地展示链路不再无条件依赖远端 `/new`
- **验证方式**: `python3 -m unittest tests/test_issue_126_standalone_http_setup_base_url.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260128-basic
  ```

### TC-002: HTTP 请求前必须校验并构造绝对地址
- **关联 AC**: AC4, AC5, AC7
- **类型**: 反向 / 边界
- **测试数据**: 无额外 testdata，源码级回归测试
- **前置条件**: 仓库可读取 `http.go`
- **测试步骤**:
  1. 检查 `HttpJsonRequest()` 是否仍直接对 `URL(path)` 调用 `http.Post`
  2. 检查是否引入 URL 解析/校验辅助逻辑
- **预期结果**: 请求发送前有显式的 base URL 合法性检查，且禁止把裸路径直接交给 `http.Post`
- **验证方式**: `python3 -m unittest tests/test_issue_126_standalone_http_setup_base_url.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260128-basic
  ```

### TC-003: 缺失或非法 base URL 走可诊断错误而非 panic
- **关联 AC**: AC1, AC5
- **类型**: 反向 / 错误处理
- **测试数据**: 无额外 testdata，源码级回归测试
- **前置条件**: 仓库可读取 `http.go`
- **测试步骤**:
  1. 检查 HTTP 初始化链路是否仍用 `C(err, "http post")` 直接升级配置错误
  2. 检查是否新增缺失协议、空地址、非法地址的诊断语义
- **预期结果**: 配置错误由上层处理为可诊断错误或降级提示，不直接 panic
- **验证方式**: `python3 -m unittest tests/test_issue_126_standalone_http_setup_base_url.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260128-basic
  ```

### TC-004: tar `--http` 回归路径仍可进入 HTTP 主链路
- **关联 AC**: AC6
- **类型**: 回归
- **测试数据**: `testdata/python/20260128-basic`（如子模块已初始化）
- **前置条件**: 可读取 `main.go`
- **测试步骤**:
  1. 检查 tar 模式是否仍通过 `mainStandalone()` / `HttpLoop()` 进入 HTTP 展示路径
  2. 后续实现完成后，对 tar 用例执行真实回归
- **预期结果**: 修复不会绕开 tar `--http` 主链路，也不会遗落该入口
- **验证方式**: `python3 -m unittest tests/test_issue_126_standalone_http_setup_base_url.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260128-basic
  ```

### TC-005: HTTP 初始化重构后项目仍可构建
- **关联 AC**: AC4, AC7
- **类型**: 回归
- **测试数据**: 无
- **前置条件**: Go 构建环境可用
- **测试步骤**:
  1. 执行 `./maze --build`
- **预期结果**: HTTP 初始化重构不破坏现有构建与入口
- **验证方式**: `python3 -m unittest tests/test_issue_126_standalone_http_setup_base_url.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260128-basic
  ```

## 说明

- 当前新增的是 **TDD 回归测试**，用于先把需求约束固定下来；在功能未实现前，相关断言预期失败。
- 动态 AC 仍需开发完成后使用真实 Python 3.11 进程与 tar 包执行验收。
- `testdata/python/20260128-basic` 目前若子模块未初始化，动态验收需先补齐 testdata 子模块内容。
