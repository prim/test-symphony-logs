## Code Review Result: FAIL

## 架构与设计
- 发现 1：`common/refs/v1` 现在把真实 RocksDB 版实现整体放到 `grocksdb_clean_link` build tag 后面，而 `common/refs/select_default.go` 与 `common/refs/select_v1.go` 仍然无条件把默认实现和 `refs_v1` 显式选择都绑定到 `maze/common/refs/v1`。这意味着“选择 v1”已经不再等价于“使用 v1 的真实实现”，而是会在未附带 `grocksdb_clean_link` 时静默落到 `refs_norocksdb.go` 的降级版本，造成版本选择语义与实现能力脱钩。

## 代码质量发现
- 发现 1：`refs_v1` / 默认 `v1` 选择语义发生静默漂移。当前 `common/refs/select_default.go`、`common/refs/select_v1.go` 都继续声明选中了 `v1`，但 `common/refs/v1/refs.go` 被 `//go:build grocksdb_clean_link` 限制后，未带该 tag 的构建实际只会编译 `common/refs/v1/refs_norocksdb.go`。这会让调用方在版本名仍为 `v1` 的前提下，拿到一个不具备 RocksDB 持久化查询能力的兼容回退实现；`GetReferents` / `GetReferrers` 永远返回 `nil` 只是把行为改得更接近当前真实实现的“无 DB”分支，但并没有解决“显式选择 v1 却未得到真实 v1”这一架构层语义错位。该问题会让构建标签、版本切换和问题定位变得混乱，后续也容易让依赖 `refs_v1` 的链路在无感知下退化。文件：`common/refs/v1/refs.go`、`common/refs/v1/refs_norocksdb.go`、`common/refs/select_default.go`、`common/refs/select_v1.go`；严重程度：major

## 建议改进
- 将“无 RocksDB 依赖时可编译的 prepare 回退需求”和“refs 版本选择”解耦：例如单独引入明确的 fallback 版本/tag，或让 `refs_v1` 明确要求真实 v1 实现，而不是复用同一包名承载两套能力模型。
- 为版本选择补一条面向外部契约的测试，校验 `CurrentVersion()`、build tags 与可用能力之间的一致性，避免后续再次出现版本名与实际实现不一致的问题。

## 总结
本轮与 issue #173 直接相关的 portable GDB 抓 core 代码整体方向正确，`prepare/corefile.go` 的 `gcore` 依赖也已移除；但为配合测试环境引入的 `common/refs/v1` build tag 改造，把“选择 v1”悄悄变成了“可能只是一个无 RocksDB 的兼容回退层”。这是版本边界和构建语义层面的 major 问题，会影响后续维护与排障，因此本次代码审查结论为 FAIL。
