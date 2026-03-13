# Test Cases

## 关联计划
Feature: Refactor MPM into facade + versioned packages

## Test Case 列表

### TC-001: 门面目录骨架与职责分层存在性检查
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 无；静态源码结构检查
- **前置条件**: 仓库存在 `common/mpm` 迁移结果
- **测试步骤**:
  1. 运行 `go test ./qa/issue134 -run TestMPMFacadePackageScaffoldExists -count=1`
  2. 检查 `common/mpm/facade.go`、`select_default.go`、`select_v1.go`、`select_v2.go`
  3. 检查 `common/mpm/shared/types.go` 与 `common/mpm/v1`、`common/mpm/v2` 目录存在
- **预期结果**: 四层目录与关键入口文件齐全，支持稳定门面 + 共享定义 + 版本实现分层
- **验证方式**: Go 单测静态校验
- **验收命令**:
  ```bash
  go test ./qa/issue134 -run TestMPMFacadePackageScaffoldExists -count=1
  ```

### TC-002: 业务层仅依赖 `maze/common/mpm`
- **关联 AC**: AC2
- **类型**: 反向
- **测试数据**: 无；全仓 Go import 扫描
- **前置条件**: 业务代码已完成 import 收敛
- **测试步骤**:
  1. 运行 `go test ./qa/issue134 -run TestBusinessPackagesUseFacadePackageOnly -count=1`
  2. 扫描非 `common/mpm`、非测试文件的 import 列表
  3. 断言不存在对 `maze/common/mpm/v1`、`maze/common/mpm/v2`、`maze/common/mpm/shared` 的直接依赖
- **预期结果**: 业务层只经由门面包访问 MPM，不直接依赖版本实现与 shared 包
- **验证方式**: Go AST 静态扫描
- **验收命令**:
  ```bash
  go test ./qa/issue134 -run TestBusinessPackagesUseFacadePackageOnly -count=1
  ```

### TC-003: V1/V2 导出全局函数 API 完全一致
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: 无；`common/mpm/v1` 与 `common/mpm/v2` 源码
- **前置条件**: 两个版本包均已落地
- **测试步骤**:
  1. 运行 `go test ./qa/issue134 -run TestVersionPackagesExportSameGlobalFunctions -count=1`
  2. 解析两个版本包内的导出全局函数
  3. 对比函数名与签名集合
- **预期结果**: V1/V2 对外导出函数集合完全一致，门面层可无条件绑定任一版本
- **验证方式**: Go AST 签名对比
- **验收命令**:
  ```bash
  go test ./qa/issue134 -run TestVersionPackagesExportSameGlobalFunctions -count=1
  ```

### TC-004: 门面选择不再依赖环境变量切换
- **关联 AC**: AC1, AC3
- **类型**: 反向
- **测试数据**: 无；`common/mpm` 门面文件
- **前置条件**: 门面层已实现版本绑定逻辑
- **测试步骤**:
  1. 运行 `go test ./qa/issue134 -run TestFacadeSelectionDoesNotDependOnEnvSwitch -count=1`
  2. 检查门面层关键文件内容
  3. 断言不存在 `os.Getenv`、`MAZE_MPM_V2`、`EnableMPMV2`、`MMV2` 等旧切换标记
- **预期结果**: 版本切换由门面层单点绑定或 build tags 控制，而非运行时环境变量路由
- **验证方式**: 文件内容静态检查
- **验收命令**:
  ```bash
  go test ./qa/issue134 -run TestFacadeSelectionDoesNotDependOnEnvSwitch -count=1
  ```

### TC-005: 不引入双全局状态，并保留 V2 关键能力验证入口
- **关联 AC**: AC4, AC5
- **类型**: 边界
- **测试数据**: 无；`common/mpm/v2` 源码与配套测试文件
- **前置条件**: 新结构已包含 race / benchmark / facade contract 测试
- **测试步骤**:
  1. 运行 `go test ./qa/issue134 -run TestV2CriticalBehaviorTestsExist -count=1`
  2. 检查 V2 源码中仍存在跨页 piece、IterPage、known 位并发相关能力标记
  3. 检查 `common/mpm/facade_contract_test.go`、`common/mpm/v2/race_test.go`、`common/mpm/benchmark_smoke_test.go` 已创建
- **预期结果**: 新结构未丢失 V2 关键特性，且有直接针对门面/API/race/benchmark 的验证文件
- **验证方式**: 静态源码与测试文件存在性检查
- **验收命令**:
  ```bash
  go test ./qa/issue134 -run TestV2CriticalBehaviorTestsExist -count=1
  ```

## 补充验收建议（供开发完成后执行）

### MPM 直接相关验证
1. `go test ./common/mpm/...`
2. `go test -race ./common/mpm/...`
3. `go test ./common/mpm -run '^$' -bench 'Benchmark' -benchmem`

### testdata 回归建议
1. `python3 testdata/run_test.py python/20260129-complex-types-311`
2. `python3 testdata/run_test.py cpp/20260304-stack-multithread`
3. `python3 testdata/run_test.py nodejs/20260225-maze-vs-heapsnapshot`

## Test File(s)
- `qa/issue134/mpm_facade_migration_test.go`: 验证 MPM 门面迁移的目录结构、依赖边界、API 对齐与关键测试入口

## 备注
- 当前测试采用 TDD 方式，依赖新目录与新测试文件落地后才会通过。
- 由于需求要求保持可编译、可回滚、可验证，测试重点放在“结构正确性 + 约束不回退”而非实现细节耦合。
