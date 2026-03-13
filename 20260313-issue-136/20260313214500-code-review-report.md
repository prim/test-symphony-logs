## Code Review Result: FAIL

## 架构与设计
- 本次将 prepare 结果收敛到 `common.S` / `common.Cpp` 的方向是正确的，整体上减少了 `Context` 与全局结果之间的镜像同步，符合单一真实来源的设计目标。
- 但当前收敛并不完整：部分被删除的 `Context` 结果字段已经失去唯一写入路径，导致 `common.S` 中对应字段只能保留零值。该问题说明“删除镜像字段”和“迁移写入责任”没有做到一一配套，收敛过程存在遗漏。

## 代码质量发现
- 发现 1：`Tcmalloc` 的部分收敛字段在删除 `Context` 镜像后没有迁移到新的单一写入路径，导致 `S.Tcmalloc.Code`、`S.Tcmalloc.ClassToSize` 永远无法在 prepare 阶段被填充。下游 `mallocer/tcmalloc/init.go` 仍依赖这两个字段决定 size class 初始化逻辑，因此重构后会把历史上可用的 tcmalloc 元数据静默降级为零值路径，存在行为退化风险。位置：`prepare/context.go`、`prepare/fill.go`、`prepare/alloc.go`、`mallocer/tcmalloc/init.go`。严重程度：major
- 发现 2：`Mimalloc` 与项目特定模块中也存在同类“删字段但未迁移写入”的问题。当前仓库中找不到对 `S.Mimalloc.Miversion`、`S.Asiocore.Revision`、`S.Mosd.Revision` 的赋值；同时本轮范围内的 `S.Mimalloc.Version` / `S.Mimalloc.Sversion` 也未见新的写入路径。由于这些字段已从 `prepare/context.go` 删除、`FillCommonVars()` 也不再同步，prepare 结束后它们只能保持默认值。即使 QA 用例覆盖了结构收敛，这些结果字段也已经不再满足“直接写入 `S` / `Cpp` 且导出兼容”的目标。位置：`prepare/context.go`、`prepare/fill.go`、`prepare/alloc.go`，以及下游读取点 `mallocer/mimalloc/mimalloc.go`。严重程度：major

## 建议改进
- 为本轮收敛字段建立“字段级迁移清单”，逐项核对三件事：旧写入点是否删除、`S/Cpp` 新写入点是否存在、下游消费点是否仍可拿到非零值。不要只验证“结构体字段删除”和“FillCommonVars 不再搬运”。
- 为 `prepare` 增加更强的静态/单测约束：不仅检查残留引用，还要检查被列入范围的字段在 prepare 代码中至少存在一个写入路径，避免再次出现“删除成功但数据丢失”的回归。
- 如果 `GDB.ToJson*` 仅用于兼容快照或调试缓存，建议后续明确其生命周期与职责，避免再次演化出新的隐式结果来源。

## 总结
本次重构方向正确，但当前实现遗漏了多组字段的实际写入迁移，导致部分 prepare 结果在运行后只能保持零值，属于会影响下游分析行为的数据退化问题。由于存在 major 级别缺陷，本次代码审查结论为 FAIL。
