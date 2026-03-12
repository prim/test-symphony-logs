## Test Cases Designed

### Test File(s)
- `tests/test_issue_118_prepare_merge_regression.py`: 覆盖 Go prepare 合并后暴露出的构建阻断与默认测试入口阻断问题

### Cases
1. `test_prepare_corefile_manager_defines_gcore_when_prepare_invokes_it`: 验证 `prepare` 在调用 `CorefileManager.Gcore` 时，仓库内确实存在对应方法实现，避免出现未定义符号导致编译失败。
2. `test_maze_build_succeeds_after_prepare_merge`: 验证 `./maze --build` 可成功完成，覆盖合并后的基础构建链路。
3. `test_legacy_python_style_maze_invocation_remains_compatible_for_test_infra`: 验证旧测试基础设施常见的 `python3 maze ...` 调用方式不会立即触发 `SyntaxError`，覆盖默认测试入口兼容性。

## Notes for Developer
- 本组测试故意从“外部可观察行为”与“最小静态约束”两侧同时覆盖问题，既能指出构建失败，也能给出更接近根因的断言。
- 第 3 个用例用于锁定回归现象：默认测试链路不能因为入口脚本类型切换而直接崩溃。修复可以通过兼容层或统一更新测试调用入口实现，但最终必须让该回归测试通过。
- 当前工作树中 `testdata/` 子模块为空，因此这里优先补充仓库内自动化测试，先把阻断问题固化为回归测试。
