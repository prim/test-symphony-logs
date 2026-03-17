## Code Review Result: FAIL

## 架构与设计
- 发现重大问题：`common/refs/v2/refs.go` 基本整份复制了 `common/refs/v1/refs.go` 的业务层实现，仅在局部替换底层 DB 调用。这样虽然表面上保留了 `openDB / writeIndexBatch / writeReverseBatch / rangeScan` 四个薄适配函数，但实际把缓存、分片、异步写入、Save/Check 生命周期、编码协议等稳定语义再次复制到 v2，导致后续任何对 refs 核心行为的修复都需要在 v1/v2 双份维护，已经违背了“仅替换存储后端、留出清晰边界”的设计目标。

## 代码质量发现
- 发现 1：`refs v2` 通过整文件复制 `v1` 逻辑来接入 BitalosDB，未把稳定的 refs 业务语义与存储实现真正分层，造成大面积重复代码和后续演进分叉风险。当前 `common/refs/v2/refs.go` 与 `common/refs/v1/refs.go` 在缓存、分片、批处理、Save/Check、编码等核心逻辑上高度重复，已经出现注释/细节轻微漂移，后续很容易出现“一边修复、另一边遗漏”的维护事故。文件：`common/refs/v2/refs.go`、`common/refs/v1/refs.go`。严重程度：major
- 发现 2：`Save()` 在未持有 shard 锁的情况下直接摘走并置空 `X.Index`，同时 `AddIndex()`/`addReverseIndex()` 仍会并发读写这些 map；`AddIndex()` 还默认假设 `X.Index` 一定非 nil。也就是说，一旦调用方在索引写入尚未完全停止时触发 `Save()`，这里会出现竞态窗口，轻则读写不一致，重则触发对 nil map 的写入 panic。v2 在新实现中完整复制了这套生命周期约束但没有任何显式保护、状态机或注释约束，属于隐藏的并发安全问题。位置：`common/refs/v2/refs.go:263-279`、`common/refs/v2/refs.go:331-367`、`common/refs/v2/refs.go:370-397`。严重程度：major

## 建议改进
- 将 refs 的稳定业务层（缓存、分片、任务队列、Save/Check 生命周期、编码协议）抽回共享层，v1/v2 只保留真正的存储适配实现，至少把“打开/关闭、批量写、范围读”之外的逻辑去重。
- 为 `Save()` 与增量写入建立明确的生命周期约束：要么在实现层加锁/状态切换保护，要么把“停止写入后才能 Save”做成显式协议并在代码中强制校验，而不是依赖隐含调用顺序。
- 补充面向存储语义的单元测试，而不是仅做结构/字符串断言；至少覆盖 batch 可见性、Flush/Close 后查询一致性、并发 Add + Save 的安全性。

## 总结
本轮实现的功能路径已经通过 QA，但从代码质量角度看仍不宜通过。最主要的问题不是功能是否可用，而是 `refs v2` 把 `v1` 的整套核心逻辑再次复制了一份，并且连带复制了脆弱的并发生命周期处理方式。这样会显著放大后续维护成本和回归风险，因此本次代码审查结论为 FAIL。
