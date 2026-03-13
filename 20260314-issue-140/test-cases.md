# Test Cases

## 关联计划
Feature: Refactor refs into facade + versioned packages

## Test Case 列表

### TC-001: refs 门面目录骨架落地
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 无，静态代码结构检查
- **前置条件**: 工作区包含 issue #140 的实现代码
- **测试步骤**:
  1. 运行 `go test ./qa/issue140 -run TestRefsFacadePackageScaffoldExists -count=1`
  2. 检查 `common/refs`、`common/refs/v0`、`common/refs/v1` 与兼容目录文件是否存在
- **预期结果**: `common/refs` 门面层与 v0/v1 目录骨架完整落地
- **验证方式**: 自动化测试
- **验收命令**:
  ```bash
  go test ./qa/issue140 -run TestRefsFacadePackageScaffoldExists -count=1
  ```

### TC-002: 默认版本绑定与双 tag 冲突保护
- **关联 AC**: AC2
- **类型**: 正向/反向
- **测试数据**: 无，静态代码结构检查
- **前置条件**: `common/refs/select_*.go` 已创建
- **测试步骤**:
  1. 运行 `go test ./qa/issue140 -run 'TestRefsVersionSelectionFilesUseBuildTags|TestRefsFacadeExportsUniformGlobalFunctions' -count=1`
  2. 检查默认构建文件是否绑定 `v1`
  3. 检查 `refs_v0 + refs_v1` 双 tag 冲突文件是否存在并包含编译期保护符号
- **预期结果**: 默认构建显式绑定 `v1`；双 tag 同时启用会在编译期失败；v0/v1 暴露一致 API
- **验证方式**: 自动化测试
- **验收命令**:
  ```bash
  go test ./qa/issue140 -run 'TestRefsVersionSelectionFilesUseBuildTags|TestRefsFacadeExportsUniformGlobalFunctions' -count=1
  ```

### TC-003: common 仅保留 refs 兼容转发层
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: 无，静态代码结构检查
- **前置条件**: `common/refs.go` 已完成重构
- **测试步骤**:
  1. 运行 `go test ./qa/issue140 -run 'TestCommonRefsBecomesThinCompatLayer|TestBusinessPackagesAvoidDirectVersionImports' -count=1`
  2. 检查 `common/refs.go` 是否导入兼容转发层
  3. 检查 `common/refs.go` 中是否仍残留主实现类型、后台队列与 RocksDB 相关函数
  4. 检查业务包未直接导入 `common/refs/v0`、`common/refs/v1`
- **预期结果**: `common/refs.go` 仅承担薄转发职责，业务层不依赖具体版本实现
- **验证方式**: 自动化测试
- **验收命令**:
  ```bash
  go test ./qa/issue140 -run 'TestCommonRefsBecomesThinCompatLayer|TestBusinessPackagesAvoidDirectVersionImports' -count=1
  ```

### TC-004: Node.js 与 txos 现有 refs 主链路不变
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: `testdata/nodejs/20260211-02-collections`
- **前置条件**: `testdata` 子模块已初始化；项目可编译
- **测试步骤**:
  1. 运行 `go test ./qa/issue140 -run TestNodejsAndTxosStillUseStableRefsEntryPoints -count=1`
  2. 执行 `./maze --build`
  3. 执行 `python3 testdata/run_test.py nodejs/20260211-02-collections`
- **预期结果**: Node.js/txos 代码仍走稳定 refs 入口，Node.js 回归用例通过
- **验证方式**: 静态检查 + testdata 自动化验证
- **验收命令**:
  ```bash
  python3 testdata/run_test.py nodejs/20260211-02-collections
  ```

### TC-005: Python 历史轻量 refs 实现抽离到 common/refs/v0
- **关联 AC**: AC5
- **类型**: 正向
- **测试数据**: `testdata/python/20260129-complex-types-311`
- **前置条件**: `testdata` 子模块已初始化；Python 历史 refs 存储实现已迁出
- **测试步骤**:
  1. 运行 `go test ./qa/issue140 -run TestPythonHistoricalRefsMovedOutOfPythonPackage -count=1`
  2. 执行 `./maze --build`
  3. 执行 `python3 testdata/run_test.py python/20260129-complex-types-311`
- **预期结果**: `python/globals.go` 与 `python/traverse.go` 不再保留历史 refs 存储实现；Python 回归用例通过
- **验证方式**: 静态检查 + testdata 自动化验证
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-006: refs 直接测试与回归基线就绪
- **关联 AC**: AC6
- **类型**: 正向/边界
- **测试数据**: `testdata/python/20260129-complex-types-311`、`testdata/nodejs/20260211-02-collections`
- **前置条件**: `common/refs/...` 测试文件已添加；`testdata` 子模块已初始化
- **测试步骤**:
  1. 运行 `go test ./qa/issue140 -run TestRefsDirectTestsAndRegressionTargetsExist -count=1`
  2. 运行 `go test ./common/refs/...`
  3. 依次执行 Python 与 Node.js 回归用例
- **预期结果**: `common/refs/...` 直接测试可运行；至少一组 Python/Node.js testdata 回归通过
- **验证方式**: 自动化测试
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

## Notes for Developer

- 参考 `qa/issue134` 的 MPM 门面化测试风格，refs 版本选择和兼容层边界建议保持同类约束。
- `python` 首轮应以“抽离存储实现但不强切行为”为原则，避免把结构整理与行为变更绑在一起。
- 当前工作树中 `testdata` 子模块为空，AC6 相关动态回归在真正实现前还需要先补齐测试数据基础设施。
