## Test Cases Designed

### Test File(s)
- `qa/issue164/refs_v2_bitalosdb_test.go`: 针对 refs v2 基于 BitalosDB 的结构落位、版本选择、CLI 参数支持、hooks 命名泛化与存储适配层约束进行静态 QA 校验。

### Cases
1. `TestRefsV2ScaffoldAndBuildSelectionExist`: 验证 `common/refs/v2/refs.go` 与 `common/refs/select_v2.go` 已创建，且 `select_v2.go` 将门面正确绑定到 `v2` 实现。
2. `TestRefsBuildTagSelectionSupportsV2AndKeepsDefaultV1`: 验证默认版本仍为 `v1`，同时 `select_default.go`、`select_v0.go`、`select_v1.go` 与 `select_conflict.go` 已扩展到支持并约束 `refs_v2`。
3. `TestMazeCLIRecognizesRefsV2`: 验证 `maze` 脚本能够识别 `--refs v2` / `refs_v2`，并在构建时追加正确的 build tag。
4. `TestRefsHooksUseGenericDBPathNaming`: 验证 `common/refs/shared/types.go` 已新增中性字段 `RefsDBPath`，并保留 `RocksDBPath` 兼容字段，同时 `common/refs.go` 已开始收敛到中性命名。
5. `TestRefsV2UsesBitalosDBWithThinStorageAdapter`: 验证 `refs/v2` 使用 `BitalosDB` 而非 `grocksdb`，并具备 `openDB` / `writeIndexBatch` / `writeReverseBatch` / `rangeScan` 等薄适配函数，保持对外合约不变。

## Notes for Developer
- 这些测试故意以“结构与约束先落位”为导向，在功能实现前应当失败，符合 TDD 预期。
- 当前测试主要覆盖 issue 描述中明确列出的 AC：`refs_v2` 版本包、`--refs v2`、默认仍为 `v1`、多 tag 冲突保护、`RefsDBPath` 泛化、BitalosDB 后端替换。
- 若后续实现采用不同文件拆分方式，请确保这些关键语义仍然可以从源码中静态验证到，否则需要同步更新测试。
