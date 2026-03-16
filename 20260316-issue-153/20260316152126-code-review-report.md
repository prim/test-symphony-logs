## Code Review Result: FAIL

## 架构与设计
- `postman-db -> db` 的主路径迁移在 Go、Python/GDB 与文档层面基本收口，路径常量下沉到公共入口的方向是对的。
- 但回归保障层存在明显设计缺口：新增的兼容性验证主要依赖“伪 legacy 夹具 + 源码字符串断言”，无法稳定证明真实历史输入兼容性，导致迁移的长期可维护性不足。

## 代码质量发现
- 发现 1：`qa/issue110/prepare_migration_test.go` 所谓的 legacy 基线并不是真正的历史实现，而是通过 `tmp/legacy-base/maze` 再次执行当前仓库里的 `REPO_ROOT/maze`。这意味着“旧目录输入兼容”回归实际上是在拿当前实现与当前实现生成的快照做比对，无法有效发现对真实 legacy 行为的回归；一旦当前实现的兼容转换逻辑本身出错，测试仍可能继续通过，形成错误安全感。文件：`tmp/legacy-base/maze`、`qa/issue110/prepare_migration_test.go`、`prepare/prepare.go`。严重程度：major
- 发现 2：新增的 `qa/issue153/db_path_migration_test.go` 大量使用源码文本包含关系来断言迁移完成，且部分断言依赖注释/字面量而非可执行行为。例如 `TestPrepareOutputDefaultsToDBButKeepsLegacyCompatibilityMarkers` 通过注释中的 `./db/%s-%s-%s` 即可通过，`TestPythonScriptsRemainParseableForCompatibility` 也没有真正解析 Python 语法。对于一次跨 Go/Python/GDB 的路径迁移，这类测试过于脆弱，重构时极易出现“行为已坏但字符串仍在”的假阳性。文件：`qa/issue153/db_path_migration_test.go`。严重程度：major

## 建议改进
- 将 legacy 兼容回归改为基于真实历史样本或冻结基线产物，而不是通过当前实现包装出“伪 legacy”结果。
- 为路径迁移补充少量行为级测试：直接调用 `BuildOutputDir`、`prepare/process` 的路径解析逻辑，以及针对 tar/layout 的真实文件结构断言，减少对源码字符串匹配的依赖。
- 如果确实需要保留静态检查，建议把它们降级为补充测试，并与行为测试明确分层。

## 总结
本次迁移主实现方向基本正确，但新增回归保障存在结构性问题：一组测试并未验证真实 legacy 兼容性，另一组测试主要验证“源码里有没有字符串”，而不是实际行为。由于这些问题会直接削弱后续维护阶段对目录迁移回归的发现能力，当前代码审查结论为 FAIL。
