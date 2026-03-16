# Test Cases

## 关联计划
Feature: prim/maze#156 Refactor refs API：删除 Python 专用通道并统一到普通 refs API

## Test Case 列表

### TC-001: common 兼容层仅暴露统一 refs API
- **关联 AC**: AC1 删除 Python 专用 refs API；AC3 facade/compat 层删除 Python 专用槽位和转发逻辑
- **类型**: 反向
- **测试数据**: 新增 `qa/issue156/refs_api_unification_test.go`
- **前置条件**: 仓库可执行 `go test`
- **测试步骤**:
  1. 检查 `common/refs.go` 的导出入口。
  2. 断言仅保留统一 refs API，且不存在 `Python*` 专用入口。
- **预期结果**: `common/refs.go` 只保留普通 refs API 转发，不再暴露 Python 专用 refs API。
- **验证方式**: `go test` 静态源码契约测试
- **验收命令**:
  ```bash
  go test ./qa/issue156 -run TestCommonRefsOnlyExposeUnifiedAPI
  ```

### TC-002: facade/compat 层删除 Python 专用槽位
- **关联 AC**: AC2 删除 `PythonResetRefs()`，统一依赖 `Init()`；AC3 删除 facade/compat 层中的 Python 专用 refs 槽位和转发逻辑
- **类型**: 反向
- **测试数据**: 新增 `qa/issue156/refs_api_unification_test.go`
- **前置条件**: 仓库可执行 `go test`
- **测试步骤**:
  1. 检查 `common/refs/facade.go`、`common/refs/compat_bridge.go`、`common/refs/compatcommon/compat.go`、`common/refs/facade_contract_test.go`。
  2. 验证这些文件中不再出现 Python 专用 refs API 名称。
  3. 验证 `InitCompat()` 不再调用 `PythonResetRefs()`。
- **预期结果**: facade 和 compat 层不再维护 Python 专用 refs 状态与函数槽位。
- **验证方式**: `go test` 静态源码契约测试
- **验收命令**:
  ```bash
  go test ./qa/issue156 -run TestFacadeAndCompatBridgeDropPythonSpecificHooks
  ```

### TC-003: refs 版本选择切换为单一全局版本
- **关联 AC**: AC4 删除默认 `v1+py-v0` 绑定，改为单一全局版本
- **类型**: 正向
- **测试数据**: 新增 `qa/issue156/refs_api_unification_test.go`
- **前置条件**: 仓库可执行 `go test`
- **测试步骤**:
  1. 检查 `common/refs/select_default.go`、`select_v0.go`、`select_v1.go`。
  2. 验证默认版本字符串为 `v1`，显式版本分别为 `v0` 和 `v1`。
  3. 验证 `select_default.go`/`select_v1.go` 不再导入 `v0` 作为 Python 通道。
  4. 验证三个选择文件都不再绑定 `python*Impl` 槽位。
- **预期结果**: refs 运行时只存在单一版本，不再出现混合绑定。
- **验证方式**: `go test` 静态源码契约测试
- **验收命令**:
  ```bash
  go test ./qa/issue156 -run TestRefsVersionSelectionUsesSingleBackendVersion
  ```

### TC-004: v0/v1 实现仅暴露统一公共合约
- **关联 AC**: AC1 删除 Python 专用 refs API；AC4 保留全局版本切换能力
- **类型**: 边界
- **测试数据**: 新增 `qa/issue156/refs_api_unification_test.go`
- **前置条件**: 仓库可执行 `go test`
- **测试步骤**:
  1. 枚举 `common/refs/v0`、`common/refs/v1` 导出函数。
  2. 断言两端都保留统一合约 `AddReverseIndex`、`AddIndexes`、`AddIndex`、`GetReferrers`、`GetReferents`、`Init`、`Save`、`Check`。
  3. 断言不再导出 `ForEachReverseIndex`、`Stats`、`ResetRefs` 等历史 reverse-index API。
- **预期结果**: v0/v1 实现只暴露统一 refs 后端接口，不泄漏历史内部通道。
- **验证方式**: `go test` 静态源码契约测试
- **验收命令**:
  ```bash
  go test ./qa/issue156 -run TestRefsImplementationsExposeOnlyUnifiedContract
  ```

### TC-005: Python 主链路改用普通 refs API 并移除 LevelDB 导出逻辑
- **关联 AC**: AC2 删除 `PythonResetRefs()`；AC5 删除 `python/traverse.go` 中基于 reverse index 的 LevelDB 导出逻辑；AC6 Python 分析结果在 `refs=v0/v1` 下均应正确
- **类型**: 正向
- **测试数据**: 新增 `qa/issue156/refs_api_unification_test.go`
- **前置条件**: 仓库可执行 `go test`
- **测试步骤**:
  1. 检查 `python/refs_shim.go` 是否改为调用 `GetReferrers` 和普通反向索引入口。
  2. 检查 `python/python.go` 初始化阶段是否移除 `PythonResetRefs()`。
  3. 检查 `python/traverse.go` 是否删除 `GenerateLevelDBFile` 与 reverse-index 导出逻辑。
- **预期结果**: Python 模块不再依赖 Python 专用 refs API，也不再自行决定 reverse index 的导出。
- **验证方式**: `go test` 静态源码契约测试
- **验收命令**:
  ```bash
  go test ./qa/issue156 -run TestPythonPackageUsesGenericRefsEntrypoints
  ```

### TC-006: 文档与历史 QA 契约同步更新
- **关联 AC**: AC1 删除 Python 专用 refs API；AC4 删除混合绑定；兼容性注意中的外部依赖排查提示
- **类型**: 反向
- **测试数据**: 新增 `qa/issue156/refs_api_unification_test.go`
- **前置条件**: 仓库可执行 `go test`
- **测试步骤**:
  1. 检查 `doc/refs.md` 是否仍描述 Python 专用 refs 设计、`pyo` 命名和混合绑定优势。
  2. 检查 `qa/issue144/refs_param_name_test.go` 是否仍保留 Python 专用 fast path 的旧契约测试。
- **预期结果**: 文档与旧 QA 契约同步反映统一后的 refs API 设计。
- **验证方式**: `go test` 静态源码契约测试
- **验收命令**:
  ```bash
  go test ./qa/issue156 -run TestDocumentationAndLegacyQAExpectationsUpdated
  ```

## Notes for Developer

- 本轮只新增 QA 契约测试，不实现功能；这些测试当前失败是符合预期的。
- 测试重点覆盖公共 API 面、版本绑定、Python 主链路迁移、历史 reverse-index 导出逻辑删除，以及文档/旧契约同步更新。
- 由于需求核心是 API 收敛与职责下沉，本轮优先使用源码级契约测试；功能实现完成后，应补充至少一个 Python `testdata/run_test.py` 动态回归用例验证 `refs=v0` 与 `refs=v1` 主链路结果一致。
