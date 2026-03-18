## Code Review Result: FAIL

## 架构与设计
- `refs_v2` 这个 build tag 目前不只选择 `common/refs` 的门面实现，还会改写 `common/refs/v1` 包本身的语义：`common/refs/v1/refs_refs_v2.go` 在 `refs_v2` 打开时把 `GetReferents` / `GetReferrers` 变成空结果、把 `Save` / `Check` 变成空实现。这让一个“选择 v2”的标签顺带把 v1 包降级成测试桩，破坏了版本包边界，也会掩盖后续针对 v1 的真实回归问题。
- `refs_v2` 在未启用 `GOEXPERIMENT=arenas` 时会通过 `common/refs/v2/noarenas.go` 正常编译并被 `common/refs/select_v2.go` 绑定为当前实现，但该实现的核心入口在运行期直接 panic。这里把“构建前提不满足”从编译期错误变成了运行期陷阱，设计上过于脆弱。

## 代码质量发现
- 发现 1：`common/refs/v1/refs_refs_v2.go` 用 `refs_v2` build tag 重写了 `common/refs/v1` 包的行为契约，返回空查询结果并跳过持久化/校验；这会让任何在 `refs_v2` 构建中直接导入 `maze/common/refs/v1` 的代码或测试得到“可编译但语义失真”的假实现，属于隐藏问题而不是隔离依赖。文件：`common/refs/v1/refs_refs_v2.go`。严重程度：major
- 发现 2：`common/refs/select_v2.go` 只要看到 `refs_v2` 就会绑定到 `v2`，而 `common/refs/v2/noarenas.go` 在没有 arenas 时提供的是运行期 panic 实现。这意味着绕过 `maze` 包装脚本直接执行 `go build -tags refs_v2` 仍能产出二进制，但该版本在初始化/查询/保存时会崩溃，错误暴露时机过晚，不利于维护和排障。文件：`common/refs/select_v2.go`、`common/refs/v2/noarenas.go`。严重程度：major

## 建议改进
- 将“为 `refs_v2` 路径下的测试去掉 RocksDB 依赖”的处理限定在测试层或专用测试标签中，不要借由 `refs_v2` 改写 `common/refs/v1` 正式包语义。
- 对 arenas 前提采用更显式的失效方式，例如在构建选择层直接阻止非 arenas 的 `refs_v2` 正式绑定，避免生成可编译但不可运行的实现。

## 总结
当前改动完成了 `refs v2` 的落位、CLI 入口和 BitalosDB 替换，但仍存在两个阻断性的代码质量问题：一是 `refs_v2` 标签越权修改了 `v1` 包契约，二是 `refs_v2` 在缺少 arenas 前提时会退化为运行期 panic 实现。两者都会削弱版本边界与可维护性，因此本轮代码审查结论为 FAIL。
