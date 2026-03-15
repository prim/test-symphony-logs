## Code Review Result: FAIL

## 架构与设计
- `common/vm/facade.go` 与 `common/vm/v3/vm.go` 共同引入了大量进程级可变全局状态（`GetMemory`/`Init`/`Close` 函数指针、`CurrentVMType`、`activeVM`、`hotVM`、`hotSeg*`、`hotSpan*`），但没有建立任何同步或生命周期边界。当前设计把“实现切换”和“热路径读取”耦合在同一组全局变量上，单线程基准场景下可以工作，但在真实并发访问、切换 VM 实现或关闭 mmap 资源时缺乏一致性保证，属于架构级并发安全问题。

## 代码质量发现
- 发现 1：V3 读取热路径依赖 `activeVM`、`hotVM`、`hotSeg` 及一组 `hotSeg*`/`hotSpan*` 裸全局变量，`Init()` / `Close()` / `SwitchTo*()` 会直接写这些状态，而 `GetMemory*()` 同时无锁读取并解引用其派生出的指针；这会导致数据竞争，且存在 `Close()` 关闭 mmap 后旧热缓存仍被命中继续读取的生命周期风险。文件：`common/vm/v3/vm.go`、`common/vm/facade.go`。严重程度：critical。
- 发现 2：`bindImplementation()` 会直接重绑全局函数指针 `GetMemory`、`GetMemory4`、`Init`、`Close` 与 `CurrentVMType`，`SwitchToFull()` / `SwitchToV3()` 又在无同步保护下执行“关闭旧实现 + 绑定新实现 + 初始化新实现”。如果调用方存在并发访问，可能出现读线程命中半更新函数表，或旧实现 `Close()` 与新实现读取交错执行的问题。文件：`common/vm/facade.go`。严重程度：major。
- 发现 3：V3 快路径使用 `unsafe` 直接将任意地址转换为 `*uint64` / `*uint32` 并解引用（如 `loadUint64Base()`、`loadUint32Base()`、`loadUint64Bias()`、`loadUint32Bias()` 等），没有任何对齐约束检查。当前 benchmark 明确覆盖了未对齐访问场景，因此这不是理论问题；在对未对齐访问要求更严格的平台上，这种实现存在未定义行为或崩溃风险，也削弱了该模块作为底层内存分析组件的可移植性与可维护性。文件：`common/vm/v3/vm.go`。严重程度：major。
- 发现 4：`GetMemoryBatch8()` / `GetMemoryBatch4()` 的“批量 API”实际上只是逐个调用单点读取接口，既没有按地址排序/聚合，也没有 segment 级批处理、连续读取、预取或缓存友好优化。该实现满足接口存在性，但与需求中“batch API 用于扫描优化”的设计意图不一致，后续调用方容易误以为使用批量接口就能获得显著吞吐收益。文件：`common/vm/v3/batch.go`。严重程度：major。
- 发现 5：`requireV3FasterThanV2()` 采用 5 轮 benchmark 取“最佳值”比较，并用 `v3Cost >= v2Cost` 作为严格失败条件。这种 guard 容易被偶然噪声、热身状态、CPU 调频或 CI 背景负载影响，稳定性不足；它更像开发期参考基准，而不适合作为严格的性能守门方法。文件：`common/vm/bench_compare_test.go`。严重程度：minor。

## 建议改进
- 将 facade 层与 V3 热缓存状态收敛为单一的、具备明确生命周期语义的原子快照或只读结构，避免“切换实现”和“执行热路径”共享可变全局变量。
- 为 unsafe 读取路径补充对齐策略说明；若需要兼容更广平台，可考虑将未对齐地址统一落入 `binary.LittleEndian` 慢路径，仅让已证明对齐的路径使用直接解引用。
- 若要保留 batch API，建议明确其语义边界；要么实现真正的批处理优化，要么在命名/文档中说明其当前只是便捷接口而非性能承诺。
- 性能守护测试建议改用中位数/均值加容忍区间，或仅验证不低于某一收益阈值，以减少 CI 偶发抖动带来的误报。

## 总结
本轮改动在功能覆盖和 benchmark 数量上已经较完整，但代码质量层面仍存在阻塞性问题。最关键的是：V3 通过多组无锁全局变量维护当前 VM 与热缓存状态，而 facade 层又允许在运行时无同步切换实现，这使得该方案在并发环境下缺乏可靠的线程安全与资源生命周期保证；同时，快路径对任意地址使用 `unsafe` 直接解引用，也给未对齐访问带来了平台相关风险。由于这些问题都位于底层内存读取主路径，影响范围广，建议本轮评审结论为 FAIL，由后续修复代理处理。
