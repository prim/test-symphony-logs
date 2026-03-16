## Code Review Result: FAIL

## 架构与设计
- `common/vm/facade.go` 的实现切换流程存在生命周期发布顺序问题：`switchImplementation()` 先关闭旧实现，再把新实现快照发布到全局 dispatcher，最后才调用新实现的 `Init()`。这会让并发读路径在新实现尚未完成初始化时就可见到它，也会在目标实现不存在时保留“已关闭的旧实现快照”，属于全局门面层的状态机设计缺陷。

## 代码质量发现
- `common/vm/facade.go:116-126`，严重程度 major：`switchImplementation()` 在 `bindImplementation()` 之后才执行 `Init()`，而 `dispatchGetMemory/GetMemoryRange` 等读入口并不受 `lifecycleMu` 保护，其他 goroutine 可以在新 VM 尚未初始化完成时直接读到新实现；同时当 `bindImplementation(t)` 失败时，函数直接返回，但旧实现已被 `Close()`，全局快照仍指向旧实现，后续调用会继续命中已关闭对象。
- `common/vm/v3/vm.go:277-316,564-588,765-783`，严重程度 critical：`Segment.ensureLoaded()` 使用 `sourceHandle.gen` 协调懒加载，但发布缓存时并未校验 `segmentData.gen` 是否仍与当前 loader 代际一致。`bindSource()` / `release()` 会清空 `seg.data` 并重置 `loading`，旧 goroutine 仍可能在之后把旧 source 读取到的 `buf` 重新 `Store` 回 segment，导致 source 切换或关闭后重新暴露陈旧数据，属于并发代际失效问题。
- `common/vm/v3/vm.go:498-527,417-429,1158+`，严重程度 critical：`linearSpan.init()` 直接设置 `limit8 = last.End - 7`、`limit4 = last.End - 3`，未处理线性 span 总长度小于 8/4 字节的情况。由于使用 `uint64`，极小 span 会发生下溢，随后 `containsUint64/containsUint32` 可能错误放行地址，进入基于 `unsafe` 或 `binary.LittleEndian` 的快速路径，产生越界读取风险。
- `common/vm/v3/batch.go:327-427`，严重程度 major：`MemoryIterator` 创建后直接缓存 segment/linear 的底层 `[]byte`、`unsafe.Pointer` 和 `[]uint64` 视图，但它没有像公共读 API 那样经过 `acquireReadState()/releaseReadState()` 生命周期保护。若后续启用真正的 `mmapSource + closer`，`Close()` 只能等待受 read-state 管理的读者，无法感知已创建的 iterator，存在关闭后继续读取已失效映射的悬垂引用风险，API 语义也与其它公共入口不一致。
- `common/vm/bench_compare_test.go:177-211,251-387`，严重程度 minor：性能守护测试 `requireV3FasterThanV2()` 以 `testing.Benchmark` 在测试进程内做 5 轮中位数比较，并允许 `v3 <= v2 * 1.05`。这既与报错文案“V3 优于 V2”不一致，也容易受 CI 机器负载和调度抖动影响，造成伪阳性或伪阴性，长期维护成本较高。

## 建议改进
- 为 facade 切换建立明确的两阶段状态机：先构造并初始化新实现，再一次性发布新快照；切换失败时应回滚到“无实现”或保留仍可用的旧实现，而不是暴露已关闭实例。
- 为 `segmentData` 的读取和写回补齐代际校验，确保旧 loader 的加载结果不会在新生命周期中重新发布。
- 将 linear fast path 与 segment fast path 统一成同一套长度/溢出校验逻辑，避免边界条件分叉。
- 为 `MemoryIterator` 明确生命周期契约：要么纳入 read-state 引用计数保护，要么文档化其仅限 VM 生命周期内使用并在类型层限制误用。
- 当前工作树存在与本 issue 无关的未提交修改：`common/utils.go`、`nodejs/map_info.go`。虽然未纳入本次审查结论，但后续提交应避免混入。

## 总结
本轮改动在性能方向上做了较大推进，但门面层切换、V3 懒加载代际一致性、线性快速路径边界处理这三处都属于长期维护中容易触发的高风险问题。其中后两项已达到 critical，可能导致关闭/切换后的陈旧数据读取或越界访问风险，因此本次代码审查结论为 FAIL，建议由 fix agent 先修复上述阻断问题后再进入下一轮审查。
