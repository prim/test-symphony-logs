## Test Cases Designed

### Test File(s)
- `tests/test_issue_97_move_core_modules_to_maze_py.py`: 验证 4 个基础模块迁移到 `maze-py/`、`.maze.py` 的路径注入顺序、旧专项测试的上下文适配，以及模块可从 `maze-py/` 成功导入。

### Cases
1. `test_core_modules_are_moved_out_of_repository_root`: 验证根目录不再保留 `maze_globals.py`、`maze_collect.py`、`maze_db.py`、`maze_utils.py`，且对应文件存在于 `maze-py/`。
2. `test_maze_py_bootstraps_maze_py_before_importing_core_modules`: 验证 `.maze.py` 在导入核心模块前，使用基于 `__file__` 的绝对路径和 `sys.path.insert(0, ...)` 注入 `maze-py/`，并移除旧的 `append` 逻辑。
3. `test_issue_85_and_89_tests_bootstrap_maze_py_directory`: 验证受影响的专项单元测试已在测试上下文中优先注入 `ROOT / "maze-py"`，避免继续依赖仓库根目录路径。
4. `test_core_modules_can_be_imported_from_maze_py_directory`: 验证在仅保留 `maze-py/` 为优先模块搜索路径时，4 个基础模块都能从新目录导入，覆盖迁移后的运行行为。

## Notes for Developer
- `.maze.py` 的路径注入必须发生在任何 `maze_*` 基础模块导入之前，否则迁移后会直接触发 `ImportError`。
- 路径推导必须使用 `os.path.dirname(__file__)`，不要写死 `./maze-py` 之类相对字面量。
- 迁移后请同步修复 `tests/test_issue_85_maze_py_split_phase1.py` 与 `tests/test_issue_89_extract_classes_to_maze_py.py` 的导入上下文，否则单元测试会继续从旧位置寻找模块。
