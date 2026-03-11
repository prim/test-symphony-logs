# Test Cases

## 关联计划
Feature: 继续拆分 `.maze.py` 剩余类至 `maze-py` 目录

## Test Case 列表

### TC-001: 剩余类均已拆分为独立模块
- **关联 AC**: AC1
- **类型**: 正向
- **测试数据**: 无，代码结构检查
- **前置条件**: 仓库中存在 `.maze.py` 与 `maze-py/`
- **测试步骤**:
  1. 检查 `maze-py/` 目录是否存在。
  2. 检查 `storage.py`、`corefile.py`、`symbols.py`、`lua.py`、`libc.py`、`c_lang.py`、`openblas.py`、`jemalloc.py`、`tcmalloc.py`、`mimalloc.py`、`python_lang.py`、`gdb_wrapper.py`、`process.py`、`cpp_analyzer.py`、`cpp_vtable.py` 是否存在。
- **预期结果**: 所有计划中的剩余类都有对应独立文件。
- **验证方式**: 自动化单元测试 `tests/test_issue_94_split_remaining_classes.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-002: `.maze.py` 不再保留剩余类内联定义
- **关联 AC**: AC1
- **类型**: 反向
- **测试数据**: 无，代码结构检查
- **前置条件**: `.maze.py` 可读取
- **测试步骤**:
  1. 搜索 `.maze.py` 中 `Storage/Corefile/Symbols/Lua/Libc/C/OpenBLAS/Jemalloc/Tcmalloc/Mimalloc/Python/GDB/Process/Cpp/CppVtable` 的类定义片段。
  2. 校验这些片段已全部移除。
- **预期结果**: `.maze.py` 中不再出现这些类的内联定义。
- **验证方式**: 自动化单元测试 `tests/test_issue_94_split_remaining_classes.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260304-stack-multithread
  ```

### TC-003: builtins 注入清单完整
- **关联 AC**: AC1, AC5
- **类型**: 正向
- **测试数据**: 无，代码结构检查
- **前置条件**: `.maze.py` 可读取
- **测试步骤**:
  1. 检查 `.maze.py` 中 Python 2/3 兼容的 `builtins` 导入逻辑。
  2. 检查共享函数、全局变量、单例对象是否显式注入到 `builtins`。
- **预期结果**: 新拆分模块依赖的 builtins 契约完整可用。
- **验证方式**: 自动化单元测试 `tests/test_issue_94_split_remaining_classes.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-004: 导入顺序满足依赖层级
- **关联 AC**: AC1
- **类型**: 边界
- **测试数据**: 无，代码结构检查
- **前置条件**: `.maze.py` 可读取
- **测试步骤**:
  1. 检查 `.maze.py` 中对剩余拆分模块的 `import` 顺序。
  2. 确认顺序符合 `storage → corefile → symbols → lua/libc/c_lang/openblas → jemalloc/tcmalloc/mimalloc → python_lang → gdb_wrapper → process → cpp_analyzer → cpp_vtable`。
- **预期结果**: 导入顺序与依赖层级一致，避免初始化时循环依赖问题。
- **验证方式**: 自动化单元测试 `tests/test_issue_94_split_remaining_classes.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260304-stack-multithread
  ```

### TC-005: 拆分模块无需显式导入基础运行时依赖
- **关联 AC**: AC5
- **类型**: 反向
- **测试数据**: 无，AST 检查
- **前置条件**: `maze-py/` 中目标模块存在
- **测试步骤**:
  1. 解析每个拆分模块 AST。
  2. 检查是否显式导入 `builtins/__builtin__/maze_globals/maze_utils/maze_db/maze_collect`。
  3. 检查是否显式 `from ... import singleton/once/log/run/fatal/Process/GDB`。
- **预期结果**: 所有拆分模块都通过 builtins 获得基础依赖，而不是显式导入。
- **验证方式**: 自动化单元测试 `tests/test_issue_94_split_remaining_classes.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-006: 仅靠 builtins 注入即可导入拆分模块
- **关联 AC**: AC1, AC5
- **类型**: 正向
- **测试数据**: 无，导入模拟
- **前置条件**: 允许在测试中向 `builtins` 注入桩对象
- **测试步骤**:
  1. 在测试进程中注入最小化 builtins 桩对象。
  2. 动态加载每个拆分模块。
  3. 校验模块中导出的类已通过 `@singleton` 变为实例。
- **预期结果**: 所有拆分模块在 builtins 契约满足时可独立导入成功。
- **验证方式**: 自动化单元测试 `tests/test_issue_94_split_remaining_classes.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260304-stack-multithread
  ```

### TC-007: 缺少关键 builtins 时快速失败
- **关联 AC**: AC5
- **类型**: 错误处理
- **测试数据**: 无，导入异常验证
- **前置条件**: `storage.py` 已拆分
- **测试步骤**:
  1. 临时移除 `builtins.singleton`。
  2. 尝试直接导入 `maze-py/storage.py`。
- **预期结果**: 导入过程抛出 `NameError`，说明模块明确依赖 builtins 契约，没有静默使用错误路径。
- **验证方式**: 自动化单元测试 `tests/test_issue_94_split_remaining_classes.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-008: `.maze.py` 体积达到目标
- **关联 AC**: AC2
- **类型**: 边界
- **测试数据**: 无，文本统计
- **前置条件**: `.maze.py` 可读取
- **测试步骤**:
  1. 统计 `.maze.py` 总行数。
  2. 与 1000 行阈值进行比较。
- **预期结果**: `.maze.py` 行数小于等于 1000。
- **验证方式**: 自动化单元测试 `tests/test_issue_94_split_remaining_classes.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-009: 源码保持 Python 2.7 / 3.x 兼容语法边界
- **关联 AC**: AC6
- **类型**: 兼容性
- **测试数据**: 无，AST 语法检查
- **前置条件**: `.maze.py` 与拆分模块存在
- **测试步骤**:
  1. 解析 `.maze.py` 与所有拆分模块 AST。
  2. 检查是否引入 `async def`、类型注解赋值、f-string 等明显 Python 3 专属语法。
- **预期结果**: 代码未引入破坏 Python 2.7 兼容性的明显语法。
- **验证方式**: 自动化单元测试 `tests/test_issue_94_split_remaining_classes.py`
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260304-stack-multithread
  ```

### TC-010: Python 分析回归用例通过
- **关联 AC**: AC3
- **类型**: 正向
- **测试数据**: `testdata/python/20260129-complex-types-311`
- **前置条件**: `testdata` 子模块已初始化，环境可运行 `run_test.py`
- **测试步骤**:
  1. 执行 Python 回归用例。
  2. 检查输出结果是否通过。
  3. 如失败，结合 `maze.log` 与 `maze.py.log` 排查是否为拆分类导入或 builtins 注入问题。
- **预期结果**: `python/20260129-complex-types-311` 通过。
- **验证方式**: 自动化测试命令
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-011: C++ 分析回归用例通过
- **关联 AC**: AC4
- **类型**: 正向
- **测试数据**: `testdata/cpp/20260304-stack-multithread`
- **前置条件**: `testdata` 子模块已初始化，环境可运行 `run_test.py`
- **测试步骤**:
  1. 执行 C++ 回归用例。
  2. 检查输出结果是否通过。
  3. 如失败，结合 `maze.log` 与 `maze.py.log` 排查是否为 GDB/Process/Cpp 拆分后初始化顺序问题。
- **预期结果**: `cpp/20260304-stack-multithread` 通过。
- **验证方式**: 自动化测试命令
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260304-stack-multithread
  ```

## Notes for Developer

- 本轮新增的是 TDD 风格结构测试，当前实现尚未完成时，测试失败是预期行为。
- 测试重点覆盖：模块拆分完整性、builtins 契约完整性、导入顺序、无显式基础导入、导入时 singleton 行为、源码体积目标、Python 2/3 兼容语法边界。
- 真正的功能验收仍需以 `run_test.py` 跑通 Python / C++ 回归用例为准。
