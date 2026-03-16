# Test Cases

## 关联计划
Feature: 将 postman-db 目录迁移至 db

## Test Case 列表

### TC-001: Go 常量与默认输出目录切换到 db
- **关联 AC**: AC1, AC4
- **类型**: 正向
- **测试数据**: 无需 testdata，使用源码静态检查
- **前置条件**: 仓库包含 `common/const.go`、`prepare/args.go`
- **测试步骤**:
  1. 检查 `common/const.go` 中数据库根目录常量值。
  2. 检查 `prepare/args.go` 中默认输出目录拼接逻辑。
  3. 确认默认路径已切换到 `./db/<project>-<user>-<profileId>`。
- **预期结果**: Go 层统一输出到 `db/`，不再默认写入 `postman-db/`。
- **验证方式**: 自动化 Go 测试 `TestDBRootConstantSwitchesToDB`、`TestPrepareOutputDefaultsToDBButKeepsLegacyCompatibilityMarkers`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-002: prepare/process 新路径生效且保留 legacy 输入兼容
- **关联 AC**: AC4, AC5
- **类型**: 正向/兼容性
- **测试数据**: 无需新增 testdata，使用源码静态检查
- **前置条件**: 仓库包含 `prepare/process.go`、`prepare/prepare.go`、`qa/issue110/prepare_migration_test.go`
- **测试步骤**:
  1. 检查 `prepare/process.go` 中 maps.gdb、elf 路径是否切换到 `db/`。
  2. 检查 `prepare/process.go` 是否仍保留 `postman-db/elf` 作为历史输入回退路径。
  3. 检查 `prepare/prepare.go` 与 `qa/issue110/prepare_migration_test.go` 是否保留 `tmp/legacy-base/postman-db` 兼容语义。
- **预期结果**: 当前输出路径统一写入 `db/`，同时旧目录输入兼容逻辑仍在。
- **验证方式**: 自动化 Go 测试 `TestProcessPathsUseDBAndKeepLegacyInputFallback`、`TestIssue110MigrationTestTargetsDBOutputAndLegacyInput`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-003: main.go 日志回收逻辑从 db 复制结果
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: 无需新增 testdata，使用源码静态检查
- **前置条件**: 仓库包含 `main.go`
- **测试步骤**:
  1. 检查 `main.go` 中临时目录下数据库根路径。
  2. 检查 maze 日志、maze.py 日志回收通配路径。
  3. 确认复制逻辑不再依赖 `postman-db/*/maze.log`。
- **预期结果**: 结果回收逻辑从 `db/*/maze.log` 与 `db/*/maze.py.log` 读取日志。
- **验证方式**: 自动化 Go 测试 `TestMainCollectsLogsFromDBDirectory`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-004: Python/GDB 工具链路径常量切换到 db
- **关联 AC**: AC2, AC6
- **类型**: 兼容性
- **测试数据**: 无需新增 testdata，使用源码静态检查
- **前置条件**: 仓库包含 `cmd/postman_prelude.py`、`cmd/maze-gdb-coredump.py`、`gdb/gdb-scripts/gdb_tool.py`
- **测试步骤**:
  1. 检查 Python 脚本中 `POSTMANDB`、`POSTMANDB_ELF` 常量值。
  2. 检查 GDB 工具中 md5、exe、core、elf 路径拼接。
  3. 确认已改为 `./db` 体系。
- **预期结果**: Python/GDB 工具链默认从 `db/` 读写，不再硬编码 `./postman-db`。
- **验证方式**: 自动化 Go 测试 `TestPythonAndGDBToolingUseDBRootConstants`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-005: .gitignore 与正式文档统一新目录规范
- **关联 AC**: AC9
- **类型**: 正向
- **测试数据**: 无需新增 testdata，使用源码静态检查
- **前置条件**: 仓库包含 `.gitignore` 与正式文档
- **测试步骤**:
  1. 检查 `.gitignore` 是否忽略 `db/`。
  2. 检查 `doc/debugging.md`、`doc/maze_py_analysis.md`、`doc/support.md` 的路径示例。
  3. 确认正式文档已切换到 `db/` 规范。
- **预期结果**: 新运行目录 `db/` 被忽略，正式文档示例以 `db/` 为准。
- **验证方式**: 自动化 Go 测试 `TestGitIgnoreAndDocsPreferDBDirectory`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-006: 代表性测试执行器能够切换到 db 产物语义
- **关联 AC**: AC8
- **类型**: 正向/回归
- **测试数据**: `python/20260129-complex-types-311`
- **前置条件**: `testdata` 子模块已初始化，且存在 `testdata/run_test.py`
- **测试步骤**:
  1. 检查 `testdata/run_test.py` 是否已适配 `db/` 产物目录。
  2. 执行代表性 testdata 用例。
  3. 验证结果文件读取不再依赖 `postman-db/`。
- **预期结果**: 代表性用例在新目录规则下通过，结果可从 `db/` 正确读取。
- **验证方式**: 自动化 Go 测试 `TestRepresentativeTestdataRunnerCleansDBArtifactsAndReadsDBResults`，并使用 `run_test.py` 做动态验收
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-007: Python 工具脚本兼容性基础检查
- **关联 AC**: AC6
- **类型**: 兼容性/边界
- **测试数据**: 无需新增 testdata，使用脚本存在性与非空校验
- **前置条件**: Python 工具脚本存在
- **测试步骤**:
  1. 检查关键 Python 脚本文件存在。
  2. 检查脚本内容非空。
  3. 作为 TDD 守卫，等待开发补充 Python 2.7/3.x 实际兼容验证链路。
- **预期结果**: 关键脚本仍可作为兼容性验证入口存在，后续实现完成后应扩展为实际双版本执行验证。
- **验证方式**: 自动化 Go 测试 `TestPythonScriptsRemainParseableForCompatibility`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

## Test File(s)
- `qa/issue153/db_path_migration_test.go`: 为 issue #153 新增的 TDD 测试文件，覆盖 Go、Python/GDB、文档、忽略规则与迁移测试接线。

## Notes for Developer
- 测试按 TDD 方式编写，当前预期失败，直到实现完成目录迁移。
- `prepare/prepare.go` 中 `tmp/legacy-base/postman-db` 语义必须保留，不能机械全量替换。
- `testdata/run_test.py` 当前仓库中可能尚未初始化；动态验收依赖 `testdata` 子模块。
