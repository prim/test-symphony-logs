## Code Review Result: FAIL

## 架构与设计
- 发现 1：`common/vm/v3/vm.go:581-589` 通过全局 `vmCache` 复用同一个 `*VM`，并在 `bindSource()`（`common/vm/v3/vm.go:542-565`）中原地改写其 source、segment 数据和 linear span。这样把“缓存优化”和“实例生命周期”耦合到了一起：旧生命周期里持有的 `*VM` / iterator / batch reader 与新一次 `Init()` 会共享同一底层对象，状态边界不清晰，后续很难保证关闭、重建、并发读取之间的隔离性。严重程度：major。

## 代码质量发现
- 发现 1：`common/vm/v3/vm.go:623-640` 的 `Close()` 只清空 `defaultState/readState` 并关闭 reader，但没有清理 `vmCache`，也没有释放 `state.vm` 上已加载的 `segmentData` / `linear.buf` 引用。对于内存分析工具的大 core 场景，这会让最近一次 VM 的页表、segment 元数据以及可能已经预载入的大块内存一直保活到进程结束，关闭后内存占用无法回落，和 issue 中强调的内存开销目标相冲突。严重程度：major。
- 发现 2：`common/vm/v3/batch.go:67-79,145-219` 的 batch API 每次调用都会 `make([]batchIndex, n)` 并执行一次全量 `sort.Slice`。这会把额外分配和 `O(n log n)` 排序成本固定叠加到热路径上，尤其不利于扫描场景中本来就按地址顺序产生的批量请求，接口设计上也没有提供“调用方已排序”或“复用工作缓冲区”的途径。该实现可工作，但与“高性能版本”定位相比可维护性和可扩展性偏弱。严重程度：minor。

## 建议改进
- 将 VM 结构缓存与运行态生命周期解耦：若要保留 layout cache，建议仅缓存只读索引/元数据模板，`Init()` 时创建新的运行实例，而不是复用并原地改写旧 `*VM`。
- 在 `Close()` 中显式释放可释放引用，至少让 `vmCache`、`linear.buf/base`、各 segment 的 `data` 不再长期持有最近一次 core 的数据。
- 为 batch API 增加可复用工作区或“已排序输入”快路径，避免在扫描热点中重复分配和排序。

## 总结
本轮 V3 实现的热点路径优化、页表查址和 facade 绑定思路整体清晰，但当前把全局缓存、生命周期管理和底层数据对象耦合在同一个 `*VM` 实例上的做法，已经带来了两个阻塞性问题：一是关闭后仍会长期保留大块内存引用，二是跨生命周期复用同一对象使状态隔离变得脆弱。对于 Maze 这类面向大内存进程/大 core dump 的工具，这两点都会直接影响长期可维护性和资源安全，因此本次代码审查结论为 FAIL。
