## Test Cases Designed

### Test File(s)
- `qa/issue144/refs_param_name_test.go`: 通过 Go AST 与源码文本检查 refs 通用函数参数命名和注释是否按要求收敛。

### Cases
1. `TestGenericRefsFunctionsUseAddrParameterName`: 验证 `common/refs.go`、`common/refs/facade.go`、`common/refs/v0/refs.go`、`common/refs/v1/refs.go` 中通用函数 `GetReferrers`、`GetReferents` 以及 `rangeKeys` 的参数名统一为 `addr`。
2. `TestCommonRefsCommentsUseGenericAddressName`: 验证 `common/refs.go` 中函数注释已从 `pyo` 更新为 `addr`，避免文档与实现脱节。
3. `TestPythonSpecificFastPathMayKeepPyoName`: 验证 Python 专用函数 `PythonGetReferrersFast` 仍允许保留 `pyo` 参数名，不被误改为通用名称。

## Notes for Developer
- 本组测试只覆盖 issue 描述中明确点名的受影响函数与注释，不强制改动其他 shim 或兼容层包装函数。
- 当前测试按 TDD 预期应先失败，因为仓库内通用函数仍在使用 `pyo` 参数名，且注释尚未同步改为 `addr`。
- 若实现时选择 `objAddr` 或 `ptr`，需要同步调整测试；当前测试按 issue 推荐方案固定为 `addr`。
