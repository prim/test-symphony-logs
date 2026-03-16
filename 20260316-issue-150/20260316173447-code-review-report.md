## Code Review Result: FAIL

## 架构与设计
- `common/vm/v3` 整体分层基本清晰：模板构建、页表查址、批量读取、公共 facade 入口的职责划分是合理的。
- 但本轮实现里有两个会破坏运行时对象边界/生命周期约束的问题：一个会让模板缓存失去“只读模板”语义，另一个会让公共 fast path 绕过关闭协议。这两点都属于核心设计约束被破坏，因此不能通过。

## 代码质量发现
- 发现 1：`vmTemplate.instantiate()` 对页表做浅拷贝后，直接原地改写 `subPageTable.slots`，导致所有实例与缓存模板共享同一份二级页表对象；后续再次实例化时会把旧 VM 的页表槽位重写为新 VM 的 `Segment` 指针，破坏模板只读性，也会让已存在 VM 的查址结果指向别的实例，存在并发数据竞争和跨实例错误引用风险。位置：`common/vm/v3/vm.go:796-820`，关联结构定义在 `common/vm/v3/pagetable.go:20-29`。严重程度：critical。
- 发现 2：公共读取 fast path 仍然没有参与 inflight 计数。`GetMemory` / `GetMemorySafe` / `GetMemory4` / `GetMemory4Safe` / `GetMemoryRange` 先通过 `currentReadState()` 取到只读快照，再直接进入 `tryPublicGetMemory*` 读取底层指针；如果此时并发执行 `Close()`，关闭方可以在这些读取尚未结束时释放 mmap/segment 数据，因为 fast path 没有像 `acquirePublicReadState()` 那样登记 inflight 读者。这样会在关闭窗口内留下读取已释放底层内存的风险，对 mmap 场景尤其危险。位置：`common/vm/v3/vm.go:123-137, 1209-1289, 1338-1519`。严重程度：major。

## 建议改进
- 让模板缓存保持严格不可变：实例化时深拷贝 `subPageTable`，或改为在模板中只保存索引信息，由实例自行分配页表槽位对象。
- 将公共 fast path 纳入统一的生命周期协议：要么显式增加/释放 inflight 计数，要么采用不可释放的只读快照发布机制，确保 `Close()` 与热路径读取之间不存在悬空指针窗口。
- `runtimeState.readers` 当前基本未参与协议实现，仅在关闭时被清零，建议删除无效状态或补齐语义，避免后续维护者误判该字段已承担读者同步职责。

## 总结
本轮 V3 在性能优化方向上做了较多工作，但核心运行时安全边界还有明显缺口。尤其是页表模板的共享可变状态会直接破坏多次实例化后的对象隔离，而公共 fast path 绕过 inflight 协议则会在关闭窗口内留下潜在的悬空读取风险。两者都属于阻塞级代码质量问题，因此本次审查结论为 FAIL。
