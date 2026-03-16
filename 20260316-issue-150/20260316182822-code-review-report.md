## Code Review Result: FAIL

## 架构与设计
- `common/vm/facade.go` 将生命周期保护统一上提到 facade 层，这个方向本身是合理的，也减少了 V3 专属分支逻辑。
- 但当前切换/重初始化协议仍存在关键设计缺口：旧实现会在新实现 `Init()` 完成前被标记为 closing，导致 facade 在初始化窗口内对外变成“不可读”，没有真正做到无缝切换。

## 代码质量发现
- 发现 1：`switchImplementation()` 与 `dispatchInit()` 在调用新实现 `Init()` 之前，先对当前快照执行 `markSnapshotClosing()`；而 `acquireSnapshot()` 一旦看到 closing bit 就直接返回 `nil`。这意味着在 VM 切换或重初始化期间，所有并发 `GetMemory*` / `GetMemoryRange` 调用都会直接失败，而不是继续使用旧实现直到新实现就绪。`TestSwitchPublishesOnlyAfterInit` 只验证了 `CurrentVMType()`，没有覆盖“切换窗口内读请求仍可服务”的契约，因此这个回归未被测试捕获。对于内存分析主链路，这会把实现切换变成可见的服务中断。文件：`common/vm/facade.go:149-165, 215-230, 298-315`。严重程度：major
- 发现 2：`requireV3FasterThanV2()` 里 `if v3Cost > v2Cost || (v3Cost > noiseFloor && fasterRounds < ...)` 的判定条件前半段已经完全覆盖后半段，导致 `tolerance` 与 `fasterRounds` 实际上不会放宽任何判定，代码表达出的“允许噪声区间内波动”意图与真实行为不一致。该问题不会影响功能正确性，但会让性能守卫测试语义难以理解、难以维护，也增加后续误判风险。文件：`common/vm/bench_compare_test.go:193-216`。严重程度：minor

## 建议改进
- 为 facade 生命周期增加并发契约测试：在 `SwitchTo*()` 与 `dispatchInit()` 执行期间，验证已有旧实现仍可持续响应读取，直到新实现完成初始化后再原子切换。
- 将性能守卫判定逻辑改成与注释/变量命名一致的形式，避免 `tolerance`、`fasterRounds` 成为名义存在但实际无效的参数。

## 总结
本轮修改在“等待在途请求完成后再 Close”方面有明显改进，但 facade 生命周期治理仍未实现真正的平滑切换：一旦进入切换或重初始化窗口，对外读接口会短暂失效，这是主路径上的 major 级并发设计问题。因此本次代码审查结论为 FAIL，建议由 fix agent 继续修复后再审。
