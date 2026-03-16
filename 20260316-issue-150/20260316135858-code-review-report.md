## Code Review Result: FAIL

## 架构与设计
- 整体方向基本合理：V3 将实现选择、页表映射、惰性加载、批量读取与测试契约拆分到 `common/vm/`、`common/vm/v3/`、`common/vm/shared/`，比旧版本更清晰。
- 但当前热路径与关闭期保护、批量 API 优化之间仍有明显设计冲突：一部分“高性能”路径只在测试/小数据场景生效，进入真实大 coredump 场景后会自动退化，说明性能设计还没有完全落到主工作负载上。

## 代码质量发现
- 发现 1：V3 的公共读入口在 mmap 大文件场景下会系统性退化，导致高性能热路径无法用于真实大内存进程。`Init` 在总 segment 大于 `eagerHeapSourceLimit`（256MB）时会选择 `mmapSource` 并设置 `closer`（`common/vm/v3/vm.go:725-737`）；但 `canUseFastReadState` 明确要求 `owner.closer == nil` 才允许走无引用计数快路径（`common/vm/v3/vm.go:106-115`）。结果是 `GetMemory` / `GetMemorySafe` / `GetMemory4` / `GetMemory4Safe` / `GetMemoryRange` 在 mmap 模式下都会退回到 `acquireReadState` / `releaseReadState` 的每次读取原子加减引用计数流程（`common/vm/v3/vm.go:141-175`, `1104-1141`, `1164-1356`）。这意味着越接近 Maze 的真实使用场景（大于 256MB 的 core、大内存进程扫描），越无法享受 V3 宣称的 lock-free hot path，和需求里强调的大内存进程性能目标相冲突。文件：`common/vm/v3/vm.go`。严重程度：major。
- 发现 2：批量读取 API 在“地址已排序”这一最常见扫描输入下反而绕过了批量优化路径。`GetMemoryBatch8` / `GetMemoryBatch4` 检测到 `addresses` 已排序后，直接逐项调用 `vm.GetMemory` / `vm.GetMemory4`（`common/vm/v3/batch.go:257-266`, `282-291`）；只有输入未排序时才会先排序并进入 `readBatch8Ordered` / `readBatch4Ordered`，复用 `batchWindow` 做 segment/window 级读取优化（`common/vm/v3/batch.go:127-183`, `267-295`）。顺序扫描、heap walk、iterator 邻近访问本来就是 batch API 的核心收益场景，但当前实现把这类“最佳输入”降级成标量循环，导致 API 语义上是 batch，执行上却不是 batch，性能设计与使用习惯相背离。文件：`common/vm/v3/batch.go`。严重程度：major。

## 建议改进
- 将“关闭期安全”与“热路径读取”重新解耦，例如为 mmap reader 提供更低开销的生命周期保护策略，避免大文件场景下每次读取都做原子引用计数。
- 让 batch API 对“已排序输入”直接走窗口化批处理逻辑；未排序输入只负责补排序，不应让有序输入退化成逐项标量调用。
- 为上述两个性能敏感点补充更贴近真实场景的守护测试：一类覆盖 `>256MB` mmap 初始化后的公共入口性能路径，另一类覆盖“已排序地址批量读取”与“未排序地址批量读取”的性能对比，防止后续再次偏离设计目标。

## 总结
本轮代码在模块拆分、测试覆盖和边界修复方面已经明显完善，但仍有两处阻塞性代码质量问题：一是 mmap 大文件场景下公共读入口会整体退化，二是 batch API 在最常见的有序扫描输入下没有真正使用批量优化。这两点都直接落在 V3 的核心卖点——大内存场景性能与批量扫描优化——上，因此本次审查结论为 FAIL。
