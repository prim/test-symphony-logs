## Code Review Result: FAIL

## 架构与设计
- 本轮 `prepare/corefile.go` 统一改为通过 `ResolveGDBBin()` 生成 core，方向正确，且与 `prepare/gdb_server.go` 的职责边界基本一致。
- 但同一批改动额外把 `common/refs/v1` 的默认实现切换为无 RocksDB 回退层，这个变化超出了 issue #173“统一 GDB 抓 core”本身的范围，并引入了 `refs v1` 对外语义漂移，属于不必要的架构扩散。

## 代码质量发现
- 发现 1：`common/refs/v1/refs_norocksdb.go` 以 `!grocksdb_clean_link` 作为默认实现时，`Configure` 与 `SetMode` 被实现为纯空操作，`AddIndex` 也无条件转发到 `v0.AddIndex`。这与真实 `v1` 实现依赖 `hooks.EnablePyMerge()` 和 `mode` 控制写入/查询行为的契约不一致，会在无 RocksDB 环境下把“默认 v1”静默降级为另一套行为模型，而调用方仍通过 `common/refs/select_default.go` 认为自己拿到的是 `v1`。这类语义漂移不是测试层面的差异，而是实现层面对外契约改变，后续很容易造成 prepare 之外链路的隐性行为分叉。文件：`common/refs/v1/refs_norocksdb.go`、`common/refs/v1/refs.go`、`common/refs/select_default.go`；严重程度：major

## 建议改进
- 将“无 RocksDB 环境下让 `prepare` 定向测试可编译”继续收敛在测试入口或专用构建入口，不要让 `common/refs/v1` 在默认构建下承载与真实 `v1` 不一致的运行语义。
- 如果必须保留回退实现，至少需要显式对齐真实 `v1` 的关键行为开关（如 `Configure`/`SetMode` 对写入路径的影响），并在文档或版本选择层明确这不是完整 `v1`，避免默认 facade 继续宣称当前版本为 `v1`。
- `prepare/corefile.go` 中基于错误字符串判断 `cross-device link` 的方式较脆弱，后续可考虑改为更稳健的错误类型判断；该项目前不构成阻塞。

## 总结
`prepare` 链路统一通过 resolved GDB 抓取 core 的核心思路没有明显问题，但当前提交同时引入了 `common/refs/v1` 默认实现的语义降级，而且这种降级对外仍以 `v1` 身份暴露，破坏了实现与契约的一致性。由于该问题属于 major 级别的设计/可维护性风险，本次代码审查结论为 FAIL。
