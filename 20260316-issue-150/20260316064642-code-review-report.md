## Code Review Result: FAIL

## 架构与设计
- 发现 1：`common/vm` 门面层对 V3 的类型暴露与切换支持不完整。虽然底层已新增 `VMV3`、`SwitchToV3()` 与 `select_v3.go`，但 `common/vm.go` 仍只导出 `VMNone/VMLRU/VMFULL`，且 `SetVMType()` 仅能在 `VMLRU` 与 `VMFULL` 之间切换，导致上层调用方无法通过公共接口显式选择 V3，也让门面层 API 与底层实现能力不一致。对应位置：`common/vm/types.go`、`common/vm/facade.go`、`common/vm.go:9-13,117-132`。严重程度：major。

## 代码质量发现
- 发现 1：公共接口未完整暴露新实现，形成“已实现但不可配置”的架构割裂。当前分支已经把 V3 接入到底层注册与 build tag 选择，但 `common` 包作为项目其余模块使用的稳定入口，没有同步导出 `VMV3` 常量，也没有在 `SetVMType()` 中处理 V3 分支。这会让后续调用方、配置逻辑和文档对“支持 V3”产生误判，并增加未来继续扩展 VM 实现时的分叉维护成本。文件：`common/vm.go`、`common/vm/types.go`。严重程度：major。
- 发现 2：`common/vm/v3/vm.go` 中保留了 `loadUint64Bias` / `loadUint32Bias`、`linearSpan.range8` / `linearSpan.range4` 等未被实际读取的辅助状态与函数，说明 V3 热路径重构后仍有遗留接口未清理。它们虽然不直接影响功能，但会增加阅读成本，并让后续性能调优时更难判断哪些路径仍在生效。文件：`common/vm/v3/vm.go`。严重程度：minor。

## 建议改进
- 统一 `common` 门面层与 `common/vm` 底层的类型与切换能力，确保新增 VM 实现时只需在一处补齐导出与分发逻辑，避免再次出现能力暴露不完整的问题。
- 清理 V3 中已不再使用的 bias/range 辅助函数或字段，降低热路径代码的认知负担。
- 后续若继续增加 VM 版本，建议把 `SetVMType()` 改为显式 `switch` 分发，并对“未编译实现”的回退策略集中封装，减少 API 漏接线风险。

## 总结
本轮实现已经通过 QA 的功能与性能验收，但从代码质量角度看，V3 在公共门面层上的接线仍不完整：底层支持了 V3，公共 API 却无法显式表达和选择 V3。这属于架构一致性问题，会直接影响后续接入、配置和维护，因此本次代码审查结论为 FAIL。
