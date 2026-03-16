## Code Review Result: FAIL

## 架构与设计
- `common/vm/facade.go` 已将 VM 切换改成原子快照分发，整体方向合理，也补上了“先 Init 再发布”的时序保护。
- 但 facade 生命周期语义仍不完整：`CloseCurrent()` / `Close()` 只关闭底层实现，却不撤销当前绑定快照与 VM 类型，导致“已关闭”与“仍被视为当前实现”并存，接口契约不自洽。
- `common/vm/v3/vm.go` 中为 `closer/readers` 设计了关闭期引用保护，但当前 `Init()` 真实路径并不会建立 `shared.MmapReader`，实际生产路径与抽象设计存在脱节，增加后续维护理解成本。

## 代码质量发现
- 生命周期关闭后 facade 仍保留已关闭实现的绑定状态，文件：`common/vm/facade.go:115-126,218-225`、`common/vm/init_v3_integration_test.go:60-62`，严重程度：major。`CloseCurrent()` / `dispatchClose()` 仅调用实现的 `Close()`，没有清空 `boundImplementationSnap` 或重置 `currentVMType`。这会让 facade 在逻辑上处于“已关闭但仍绑定旧实现”的状态；测试里还需要额外手工执行 `currentVMType.Store(int32(VMNone))` 与 `boundImplementationSnap.Store(nil)` 才能构造干净状态，说明公开生命周期 API 本身不完整，容易让后续调用方误判当前 VM 是否仍可用。
- V3 关闭保护的核心抽象与真实初始化路径脱节，文件：`common/vm/v3/vm.go:141-168,746-775,777-801`、`common/vm/v3/v3_contract_test.go:698-719`，严重程度：major。当前 `runtimeState`、`acquireReadState()`、`Close()` 都围绕 `closer + readers` 做关闭期保护，但 `Init()` 中虽然保留了 `closer` 变量，实际上并未调用 `OpenMmap` 或建立任何非 nil `shared.MmapReader`；对应测试甚至明确断言大 core 场景“不应绑定 closer”。这使得生产路径主要依赖 `defaultState/readState` 置空来规避并发关闭问题，而不是依赖 facade/状态机宣称的 reader 引用保护。抽象与实现长期分叉后，后续维护者很容易误以为现有关闭协议已完整覆盖底层资源生命周期，实际并没有。

## 建议改进
- 明确 facade 的生命周期契约：关闭当前实现时，要么同步撤销当前快照并重置类型，要么把“Close 仅释放资源、不改变绑定关系”写成显式约定并补充测试覆盖关闭后的可见语义。
- 梳理 V3 的 source/closer 设计：若已决定不再走 mmap reader 路径，应删除或收敛无效抽象；若后续仍计划恢复 mmap reader，应让真实 `Init()` 路径与测试路径保持一致，避免只在测试辅助代码中验证一套生产不会触发的关闭保护。
- `common/vm/v3/pagetable.go` 里目前对每个 top entry 都会分配 `subPageTable` 容器，即使单 segment 命中也如此；这不是阻塞问题，但可进一步评估是否存在可避免的元数据开销。

## 总结
本轮修复补强了切换发布顺序、懒加载代际保护和 iterator 生命周期，方向正确；但从代码质量角度看，VM facade 的关闭语义仍然不完整，且 V3 的关闭保护抽象与真实初始化路径存在明显脱节。这两点都会直接影响后续维护者对生命周期和资源管理的理解，属于架构层面的 major 问题，因此本次代码审查结论为 FAIL。
