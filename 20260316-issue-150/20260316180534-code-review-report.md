## Code Review Result: FAIL

## 架构与设计
- `common/vm/facade.go` 已将 facade 改造成“原子快照 + dispatch”模型，方向上比直接重绑全局函数更清晰；`common/vm/v3` 也把公共入口与 benchmark 快路径分离，整体设计意图明确。
- 但当前生命周期治理仍然只对 `vm/v3` 的公共入口做了专门保护，facade 层没有形成对所有实现一致生效的关闭期协议，导致抽象边界不稳定。

## 代码质量发现
- 发现 1：facade 层在切换/关闭实现时没有等待非 V3 实现的在途读请求完成，仍可能把旧实现暴露给并发读者后立即执行 `Close`，存在生命周期竞态，文件：`common/vm/facade.go`，严重程度：major。具体表现是 `switchImplementation()` 在发布新实现前后都会直接调用旧实现的 `Close`，`CloseCurrent()` / `dispatchClose()` 也只是清空快照后立即关闭当前实现；而 `dispatchGetMemory*` 侧没有任何通用 inflight 引用计数或 epoch 保护。`common/vm/v1` / `common/vm/v2` 的 `Close` 会释放 mmap / 文件句柄，这意味着运行期若发生 `SetVMType()`、重复 `Init()` 或 `CloseCurrent()` 与并发读交错，读路径可能命中已被关闭的旧实现。当前修复只保证了 V3 绑定到公共入口时具备关闭期保护，但 facade 抽象本身仍不是线程安全的统一契约。
- 发现 2：V3 绑定逻辑依赖函数指针相等来判断“是否需要把 `Facade*` 替换成公共受保护入口”，这一契约过于隐式且脆弱，后续只要对注册入口做一层轻量包装（埋点、统计、适配器、方法值转换等），`wrapImplementationForBinding()` 就会静默失效并重新暴露未受保护的 fast path，文件：`common/vm/facade_bind_v3.go`，严重程度：major。这里的安全语义并不是通过显式字段或能力接口声明，而是通过 `reflect.ValueOf(fn).Pointer()` 猜测实现来源；这会让“关闭期保护是否生效”变成非常容易被后续重构误伤的隐藏条件，维护成本高，且失败时没有编译期或运行期告警。

## 建议改进
- 将“关闭期保护”上提到 facade 抽象层，至少为所有实现统一提供在途读引用计数/epoch 机制，而不是仅在 V3 公共入口内单独兜底。
- 用显式能力声明替代函数指针识别，例如在 `implementation` 中增加“public entry / fast path entry”双通道字段，或增加布尔/接口标记，避免未来重构时静默失去保护语义。
- 为实现切换与关闭增加并发生命周期回归测试，覆盖 `SetVMType()`、`CloseCurrent()`、重复 `Init()` 与并发读取交错的场景，而不只验证 V3 自身的关闭等待逻辑。

## 总结
本轮改动在 V3 热路径与公共保护入口分离方面有明显进展，但 facade 层仍未提供对所有 VM 实现一致成立的生命周期安全保证，同时 V3 绑定又建立在脆弱的函数指针约定之上。这两个问题都会让后续维护者在看似无害的重构后重新引入关闭期竞态，因此当前代码质量评估应为 FAIL。
