## Code Review Result: FAIL

## 架构与设计
- 发现 1：`RefsDBPath` 被 `common/refs.go` 无条件注入，而 `common/refs/core/core.go` 又始终优先使用 `RefsDBPath`，导致默认稳定版 `refs_v1` 也从原来的 `txos-rocksdb-*` 路径静默切换为 `txos-refsdb-*`。这使“仅为 v2 引入中性命名、保持 v1 稳定”的版本边界被打破，默认版本发生了额外行为漂移，不利于灰度对比、运维脚本兼容和问题定位。位置：`common/refs.go:80-95`，`common/refs/core/core.go:633-638`。严重程度：major

## 代码质量发现
- 发现 1：默认 `refs_v1` 的数据库路径命名被本轮共享 hooks 改动一并改变。`v1` 已经通过 `core.New(rocksBackend{})` 复用核心层，而核心层 `refsDBPathFunc()` 会优先读取 `RefsDBPath`；当前 `common/refs.go` 为所有版本统一注入 `TxosRefsDBFileName`，因此 `v1` 实际上不再使用 `TxosRocksDBFileName`。这不是单纯的命名清理，而是默认稳定后端的运行时产物路径变化，容易影响已有目录探测、日志检索、清理脚本以及 v1/v2 并行对照流程。位置：`common/refs.go:90-91`，`common/refs/core/core.go:633-638`，`common/refs/v1/refs.go:34-49`。严重程度：major

## 建议改进
- 将“中性路径命名”限定在 `refs_v2`，不要通过共享 hooks 直接改变默认 `refs_v1` 的外部行为；若确实要统一路径，应作为单独兼容性变更明确公告，并补齐默认版迁移测试。
- 为 `refs_v1` / `refs_v2` 分别补充运行时路径断言测试，避免以后再次出现“共享层改动意外改变默认版本行为”的问题。
- `common/refs.go` 顶部注释仍写着“支持通过 build tags 切换 v0/v1 版本”，与当前已有 `v2` 不一致，建议后续同步更新文档注释，减少维护误导。

## 总结
本轮将 refs 公共逻辑抽到 `common/refs/core` 的方向总体合理，`v2` 也具备了独立后端和薄存储适配层；但共享 hooks 的接线方式让默认稳定版 `refs_v1` 发生了未被显式控制的外部行为变化，破坏了“默认仍保持 v1 稳定、v2 作为可选后端演进”的设计边界。由于该问题影响默认版本兼容性与后续对照分析，结论为 FAIL。
