# Test Cases

## 关联计划
Feature: 将 `third_party/golib/` 移动到 `3rd/netease/golib/`

## Test File(s)
- `golib_relocation_test.go`: Go 集成测试，验证目录迁移、`go.mod` replace 更新、旧目录移除，以及在新目录布局下 `./maze --build` 仍可成功。

## Test Case 列表

### TC-001: golib 源码已迁移到 `3rd/netease/golib`
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 仓库工作树中的本地依赖目录
- **前置条件**: 仓库根目录存在 `go.mod`
- **测试步骤**:
  1. 运行 `go test ./ -run TestGolibSourceTreeMovedUnder3rdNetease`
  2. 检查 `3rd/netease/golib/go.mod`、`doc.go`、`utils/logger.go` 是否存在
- **预期结果**: 新目录存在且包含代表性 golib 源文件，说明源码已完整迁移
- **验证方式**: 自动化 Go 测试

### TC-002: `go.mod` replace 指令指向新路径
- **关联 AC**: AC2
- **类型**: 正向/配置验证
- **测试数据**: `go.mod`
- **前置条件**: 仓库根目录存在 `go.mod`
- **测试步骤**:
  1. 运行 `go test ./ -run TestGolibReplaceDirectivePointsToRelocatedPath`
  2. 断言 `go.mod` 包含 `replace git-sa.nie.netease.com/golang/golib => ./3rd/netease/golib`
  3. 断言 `go.mod` 不再保留旧路径 `./third_party/golib`
- **预期结果**: replace 指令仅指向新路径
- **验证方式**: 自动化 Go 测试

### TC-003: 原 `third_party/golib` 目录已移除
- **关联 AC**: AC4
- **类型**: 反向/清理验证
- **测试数据**: 仓库工作树目录结构
- **前置条件**: 仓库根目录存在历史目录 `third_party/`
- **测试步骤**:
  1. 运行 `go test ./ -run TestLegacyThirdPartyGolibDirectoryRemoved`
  2. 断言 `third_party/golib` 不存在
- **预期结果**: 旧目录不存在，避免后续维护时出现双份依赖
- **验证方式**: 自动化 Go 测试

### TC-004: 迁移后仍可成功执行 `./maze --build`
- **关联 AC**: AC3
- **类型**: 正向/回归
- **测试数据**: 仓库根目录与迁移后的本地依赖目录
- **前置条件**:
  - `3rd/netease/golib/` 已存在
  - `third_party/golib/` 已移除
  - `go.mod` replace 已更新
- **测试步骤**:
  1. 运行 `go test ./ -run TestMazeBuildSucceedsWithRelocatedGolib`
  2. 测试内部调用 `./maze --build`
  3. 检查 `.maze` 构建产物成功生成
- **预期结果**: 构建成功，说明迁移未破坏本地依赖引用
- **验证方式**: 自动化 Go 测试

## Edge / Failure Coverage

1. **旧新路径并存**：TC-003 会阻止开发者仅复制目录而未清理旧目录。
2. **replace 未更新**：TC-002 会阻止目录已迁移但 `go.mod` 仍引用旧路径的情况。
3. **目录不完整**：TC-001 通过检查多个代表性文件，避免只创建空目录或遗漏关键源码。
4. **构建回归**：TC-004 确认目录调整不会破坏 `./maze --build`。

## Notes for Developer

- 这些测试当前预期失败，因为仓库仍保留 `third_party/golib` 且 `go.mod` 仍指向旧路径。
- 请只修改目录布局和 `go.mod`，不要修改 `git-sa.nie.netease.com/golang/golib` 的模块声明。
- 建议在完成迁移后执行 `go test ./ -run TestGolib` 进行快速验证。
