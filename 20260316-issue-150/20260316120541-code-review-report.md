## Code Review Result: FAIL

## 架构与设计
- 发现 1：`common/vm` facade 已改为原子快照分发，解决了上一轮“全局锁进入热路径”的问题，整体方向正确；但 `VMV3` 在 `select_v3.go` 中绑定的是 `v3.FacadeGetMemory/FacadeGetMemory4/FacadeGetMemoryRange` 这组“无引用计数”的极简入口，而不是 `v3.GetMemory/...` 这组具备关闭期保护的公共入口，导致 facade 层与 V3 运行时生命周期管理出现契约错位。文件：`common/vm/select_v3.go`、`common/vm/v3/vm.go`

## 代码质量发现
- 发现 1：`select_v3.go` 把公共 VM facade 绑定到 `v3.FacadeGetMemory` 等函数，但这些函数内部只调用 `currentReadState()` 读取快照，不会像 `v3.GetMemory/GetMemory4/GetMemoryRange` 那样在 mmap 模式下通过 `acquireReadState()` 增减 reader 引用计数。与此同时，`v3.Close()` 依赖 `state.readers` 归零后才关闭 `state.closer`。这意味着经由 `common/vm` 公共入口触发的并发读取在关闭期间不会被计入活跃 reader，`Close()` 可能在它们仍访问 `span.buf` / `state.vm` 时提前关闭 mmap reader，形成关闭期 use-after-close 风险。对于内存分析工具的高频并发读场景，这是生命周期与并发安全的阻断问题。文件：`common/vm/select_v3.go`、`common/vm/v3/vm.go`（`FacadeGetMemory`、`FacadeGetMemory4`、`FacadeGetMemoryRange`、`acquireReadState`、`Close`）。严重程度：critical
- 发现 2：`common/vm/bench_compare_test.go` 新增了 `BenchmarkFacadeDispatchGetMemoryV3` / `BenchmarkFacadeDispatchGetMemoryRangeV3`，但没有任何针对“关闭期间公共 facade 读路径”的并发回归测试，因此上面的生命周期契约回归没有被现有测试捕获。就当前实现而言，性能基准补充了真实入口的热路径观测，却没有覆盖公共入口最关键的并发关闭安全约束，测试设计与架构风险点仍不匹配。文件：`common/vm/bench_compare_test.go`、`common/vm/v3/vm.go`。严重程度：major

## 建议改进
- 让 `common/vm` 在 `vm_v3` 下绑定到具备关闭期保护的公共入口，或让 `v3.FacadeGetMemory*` 在 mmap 模式下复用 `acquireReadState()`/`releaseReadState()` 的生命周期协议，而不是单纯读取快照。
- 增加“公共 facade 并发读取 + CloseCurrent/切换 VM 并发执行”的回归测试，覆盖 mmap 关闭期的 reader 计数与状态发布顺序。
- 若保留当前双层 facade 设计，建议在代码中明确区分“仅可在内部稳定快照下使用的无保护入口”和“对外公共入口”，避免后续再次误绑定。

## 总结
本轮修复已经解决上一轮关于 facade 热路径加锁、纯自旋退避和短读契约的主要问题，代码整体质量明显提升；但新的 facade 绑定方式又引入了更严重的生命周期安全缺口：`common/vm` 的公共 V3 读入口绕过了 mmap 关闭期所需的 reader 引用计数协议，导致并发关闭时存在访问已关闭底层 reader 的风险。该问题属于 critical 级别，直接影响并发安全与长期可维护性，因此本次代码审查结论为 FAIL。
