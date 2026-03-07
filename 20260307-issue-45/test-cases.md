## Test Cases Designed

### Test File(s)
- `test/test_maze_py_refactor.py`: 验证 `.maze.py` 已从超大单文件重构为模块化实现，并保留兼容入口。

### Cases
1. `test_module_package_exists`: 验证仓库中已新增承载 `.maze.py` 重构结果的模块目录。
2. `test_package_contains_multiple_modules`: 验证新目录下存在多个实际模块，而不是空目录或单文件搬运。
3. `test_legacy_entry_becomes_thin_wrapper`: 验证 `.maze.py` 已退化为薄兼容入口，不再继续保持超大单文件形态。
4. `test_legacy_entry_delegates_to_new_package`: 验证旧入口文件显式委托到新的模块化实现。
5. `test_key_responsibility_modules_exist`: 验证关键职责已被拆分到独立模块（CLI、工具、GDB、运行时/主流程）。

## Notes for Developer
- 本组测试是结构性/TDD 测试，目的是约束重构结果，而不是验证具体业务功能输出。
- 当前仓库尚未完成该重构，因此这些测试预期会失败。
- 若最终模块命名与 `maze_py` 不同，需要同步调整测试；但从可维护性和可读性看，建议保留清晰、稳定的包名。
- 建议保留 `.maze.py` 作为兼容入口文件，仅做参数转发或调用新包的 `main()`。
