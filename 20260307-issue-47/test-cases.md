# Test Cases

## 关联计划
Feature: 保持功能不变 重构 cpp package

## 测试设计依据

- 需求来源：`dev-flow/plan.md`
- 参考文档：
  - `dev-log/2026-02-25-cpp-package-naming-refactor.md`
  - `dev-log/2026-02-25-cpp-bug-hunting-tests.md`
  - `dev-log/2026-02-01-how-to-create-testcase.md`
  - `cpp/x.md`
- 现有测试风格结论：
  - 统一通过 `python3 testdata/run_test.py <type>/<case>` 执行
  - `validate.py` 只验证 `maze-result.json` 数据，不直接调用 maze
  - C++ 结果重点验证 `items[].type / amount / total_size / avg_size`
  - 类型匹配采用子串匹配，数量允许合理误差

## 测试策略

本 issue 为“**保持功能不变的 cpp package 重构**”，因此测试重点为：

1. **回归验证**：确认 cpp 包核心识别与收集路径无行为变化
2. **准确性验证**：确认类型名、数量、大小统计无明显回归
3. **边界验证**：确认空容器、深层嵌套、弱块扫描、栈局部变量等边界路径仍稳定
4. **兼容性验证**：确认 cpp 重构不破坏 Python/Node.js 相关流程，且兼容不同运行时/分配器场景
5. **性能验证**：确认中大规模 coredump 分析时间和资源占用无显著退化

## Test Case 列表

### TC-001: 全局符号与 weak malloc 路径回归
- **关联 AC**: AC1, AC2, AC3, AC4
- **类型**: 正向
- **测试数据**: `testdata/cpp/20260225-cpp-globals-and-weak`
- **前置条件**:
  - `./maze --build` 成功
  - 测试目录中存在本地 tarball 与 `validate.py`
- **测试步骤**:
  1. 执行统一测试入口运行 `globals-and-weak` 用例
  2. 检查结果中全局对象与 weak/vtable 识别是否符合预期
  3. 检查 `maze.log`、`maze.py.log` 中无 panic/traceback/异常退出
- **预期结果**:
  - 全局符号收集路径保持可用
  - weak malloc 内部扫描仍能识别目标对象
  - 结果数量与重构前基线一致或等价
- **验证方式**: 自动化测试命令 + 日志检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260225-cpp-globals-and-weak
  ```

### TC-002: vtable 对象识别回归
- **关联 AC**: AC2, AC3, AC4
- **类型**: 正向
- **测试数据**: `testdata/cpp/20260225-cpp-vtable-types`
- **前置条件**:
  - `./maze --build` 成功
  - 测试目录中存在本地 tarball 与 `validate.py`
- **测试步骤**:
  1. 运行 `cpp-vtable-types` 用例
  2. 检查首地址为 vtable 的对象是否仍被正确归类
  3. 对比类型名与对象数量是否存在明显退化
- **预期结果**:
  - vtable 检测与基于 vtable 的对象收集行为不变
  - 不出现对象漏报、类型名丢失或数量大幅下降
- **验证方式**: 自动化测试命令 + 结果比对
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260225-cpp-vtable-types
  ```

### TC-003: STL 容器分类主路径回归
- **关联 AC**: AC2, AC3
- **类型**: 正向
- **测试数据**: `testdata/cpp/20260225-cpp-containers`
- **前置条件**:
  - `./maze --build` 成功
  - 测试目录中存在本地 tarball 与 `validate.py`
- **测试步骤**:
  1. 运行 `cpp-containers` 用例
  2. 验证 vector / deque / unordered_map / list 等容器展开结果
  3. 检查容器中对象的 type/amount 统计是否满足预期
- **预期结果**:
  - 容器分类函数分发仍然正确
  - 容器内元素对象数量与类型识别结果无回归
- **验证方式**: 自动化测试命令 + 结果字段验证
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260225-cpp-containers
  ```

### TC-004: 深层嵌套与长链/指针数组回归
- **关联 AC**: AC2, AC3, AC5
- **类型**: 边界
- **测试数据**: 
  - `testdata/cpp/20260225-cpp-deep-nested`
  - `testdata/cpp/20260225-cpp-long-list-ptr-array`
- **前置条件**:
  - `./maze --build` 成功
  - 测试目录中存在本地 tarball 与 `validate.py`
- **测试步骤**:
  1. 顺序执行 `deep-nested` 用例
  2. 顺序执行 `long-list-ptr-array` 用例
  3. 检查 DFS 展开、指针追踪、深度限制相关路径是否稳定
- **预期结果**:
  - 深层嵌套结构仍可正确展开或按既有策略截断
  - 不出现死循环、panic、极端耗时
  - 指针数组/链表对象统计不退化
- **验证方式**: 自动化测试命令 + 日志检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260225-cpp-deep-nested
  python3 testdata/run_test.py cpp/20260225-cpp-long-list-ptr-array
  ```

### TC-005: 智能指针、多重继承、字符串场景回归
- **关联 AC**: AC2, AC3, AC5
- **类型**: 正向/边界
- **测试数据**:
  - `testdata/cpp/20260225-cpp-smart-ptr`
  - `testdata/cpp/20260225-cpp-multi-inherit`
  - `testdata/cpp/20260225-cpp-string`
- **前置条件**:
  - `./maze --build` 成功
  - 测试目录中存在本地 tarball 与 `validate.py`
- **测试步骤**:
  1. 顺序运行 smart-ptr / multi-inherit / string 三个用例
  2. 检查 shared_ptr/unique_ptr 指向对象、主 vtable、多种 string 场景识别结果
  3. 重点关注类型归类是否发生名称退化或计数偏差
- **预期结果**:
  - 智能指针路径保持可识别
  - 多重继承对象主 vtable 识别不变
  - string 相关对象及长字符串场景统计稳定
- **验证方式**: 自动化测试命令 + 结果字段验证
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260225-cpp-smart-ptr
  python3 testdata/run_test.py cpp/20260225-cpp-multi-inherit
  python3 testdata/run_test.py cpp/20260225-cpp-string
  ```

### TC-006: 栈局部变量收集路径回归
- **关联 AC**: AC1, AC4, AC5
- **类型**: 正向/边界
- **测试数据**: `testdata/cpp/20260304-stack-multithread`
- **前置条件**:
  - `./maze --build` 成功
  - 测试目录中存在本地 tarball 与 `validate.py`
- **测试步骤**:
  1. 运行 `stack-multithread` 用例
  2. 检查 `CollectStackLocals` 相关对象是否被正确收集
  3. 检查多线程栈帧场景下无重复计数、无异常日志
- **预期结果**:
  - 栈局部变量/参数收集路径仍可工作
  - 多线程场景下对象识别稳定，无明显重复或漏报
- **验证方式**: 自动化测试命令 + 日志检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260304-stack-multithread
  ```

### TC-007: 已知高风险容器边界回归（deque / map-set / unordered-set）
- **关联 AC**: AC2, AC3, AC5
- **类型**: 边界
- **测试数据**:
  - `testdata/cpp/20260225-cpp-deque-boundary`
  - `testdata/cpp/20260225-cpp-map-set`
  - `testdata/cpp/20260225-cpp-unordered-set`
- **前置条件**:
  - `./maze --build` 成功
  - 测试目录中存在本地 tarball 与 `validate.py`
- **测试步骤**:
  1. 顺序执行 deque-boundary 用例
  2. 顺序执行 map-set / unordered-set 用例
  3. 关注边界 iterator、红黑树/哈希桶节点遍历是否存在回归
- **预期结果**:
  - deque 首尾 buffer 边界场景结果不退化
  - map/set/unordered_set 相关对象仍能通过现有路径被识别
  - 不出现无限遍历、节点重复爆炸、结果空洞化
- **验证方式**: 自动化测试命令 + 结果字段验证
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260225-cpp-deque-boundary
  python3 testdata/run_test.py cpp/20260225-cpp-map-set
  python3 testdata/run_test.py cpp/20260225-cpp-unordered-set
  ```

### TC-008: 准确性对比非回归（Maze vs Heapsnapshot）
- **关联 AC**: AC2, AC6
- **类型**: 准确性验证
- **测试数据**: `testdata/nodejs/20260225-maze-vs-heapsnapshot`
- **前置条件**:
  - `./maze --build` 成功
  - 测试目录中存在本地 tarball 与 `validate.py`
- **测试步骤**:
  1. 执行 Maze 与 heapsnapshot 对比用例
  2. 检查对象数量/大小差异是否仍在既有容忍范围内
  3. 确认 cpp 包重构未破坏跨模块公共逻辑
- **预期结果**:
  - Node.js 准确性对比结果不劣化
  - cpp 重构不引入公共基础设施回归
- **验证方式**: 自动化测试命令 + 差异结果检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py nodejs/20260225-maze-vs-heapsnapshot
  ```

### TC-009: Python 复杂对象场景兼容性非回归
- **关联 AC**: AC6
- **类型**: 兼容性
- **测试数据**: `testdata/python/20260129-complex-types-311`
- **前置条件**:
  - `./maze --build` 成功
  - 测试目录中存在本地 tarball 与 `validate.py`
- **测试步骤**:
  1. 运行 Python 复杂对象用例
  2. 检查结果结构及日志
  3. 确认 cpp 包重构未破坏公共分析流程或结果汇总
- **预期结果**:
  - Python 分析结果保持通过
  - 无新增共享逻辑回归
- **验证方式**: 自动化测试命令 + 日志检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260129-complex-types-311
  ```

### TC-010: 大内存 C++ 进程性能基线验证
- **关联 AC**: AC5, AC6
- **类型**: 性能
- **测试数据**: 需要新建 `testdata/cpp/20260307-cpp-refactor-large-memory`
- **前置条件**:
  - 在 `testdata/cpp/20260307-cpp-refactor-large-memory/` 创建测试程序、tarball、`validate.py`
  - 测试程序构造 >1GB 的多类型对象图（vector/list/string/智能指针混合）
  - 使用 `cmd/maze-gen-coredump.py` 生成本地 tarball
- **测试步骤**:
  1. 运行统一测试入口执行大内存用例
  2. 记录总耗时、峰值内存占用、结果摘要
  3. 对比重构前基线或既定阈值
- **预期结果**:
  - 分析耗时无显著退化
  - 峰值内存占用不超过目标进程内存的 10%
  - 不出现 OOM、panic、长时间卡死
- **验证方式**: 自动化测试命令 + 手动记录性能指标
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260307-cpp-refactor-large-memory
  ```

### TC-011: jemalloc 兼容性回归验证
- **关联 AC**: AC6
- **类型**: 兼容性
- **测试数据**: 需要新建 `testdata/cpp/20260307-cpp-refactor-jemalloc`
- **前置条件**:
  - 编写 C++ 测试程序，使用 `LD_PRELOAD=3rd/jemalloc-5-3-0/lib/libjemalloc.so.2`
  - 使用 `cmd/maze-gen-coredump.py` 生成本地 tarball
  - 提供 `validate.py` 验证主要对象类型与数量
- **测试步骤**:
  1. 在 jemalloc 环境下运行统一测试入口
  2. 检查 cpp 对象识别结果与 ptmalloc 基线是否等价
  3. 检查日志中无 allocator 兼容性异常
- **预期结果**:
  - jemalloc 场景下 C++ 对象识别稳定
  - 不因 cpp 重构导致 allocator 兼容性回归
- **验证方式**: 自动化测试命令 + 日志检查
- **验收命令**:
  ```bash
  python3 testdata/run_test.py cpp/20260307-cpp-refactor-jemalloc
  ```

### TC-012: Python 2.7 / Node.js 多版本兼容性回归设计
- **关联 AC**: AC6
- **类型**: 兼容性
- **测试数据**:
  - 需要新建 `testdata/python/20260307-complex-types-py27`
  - 需要新建 `testdata/nodejs/20260307-comprehensive-node18`
  - 需要新建 `testdata/nodejs/20260307-comprehensive-node20`
- **前置条件**:
  - 分别在 Python 2.7、Node.js 18、Node.js 20 环境生成 tarball
  - 每个目录包含独立 `validate.py`
- **测试步骤**:
  1. 分别执行三个版本兼容性用例
  2. 检查核心结果与现有基线是否等价
  3. 验证 cpp 重构不会影响多语言多版本流程
- **预期结果**:
  - Python 2.7、Node.js 18/20 用例均通过
  - 无共享逻辑兼容性回归
- **验证方式**: 自动化测试命令
- **验收命令**:
  ```bash
  python3 testdata/run_test.py python/20260307-complex-types-py27
  python3 testdata/run_test.py nodejs/20260307-comprehensive-node18
  python3 testdata/run_test.py nodejs/20260307-comprehensive-node20
  ```

## 覆盖矩阵

| AC | 覆盖的 TC |
|----|-----------|
| AC1 | TC-001, TC-006 |
| AC2 | TC-001, TC-002, TC-003, TC-004, TC-005, TC-007, TC-008 |
| AC3 | TC-001, TC-002, TC-003, TC-004, TC-005, TC-007 |
| AC4 | TC-001, TC-002, TC-006 |
| AC5 | TC-004, TC-005, TC-006, TC-007, TC-010 |
| AC6 | TC-008, TC-009, TC-010, TC-011, TC-012 |

## 说明

1. 本次 issue 属于“重构不改行为”，因此验收主线是**回归测试**而不是新增功能测试。
2. C++ 现有高价值回归集应优先执行：`globals-and-weak`、`containers`、`smart-ptr`、`stack-multithread`。
3. `TC-010 ~ TC-012` 为补足 issue 所要求的性能/兼容性覆盖而设计的新用例；若进入执行验收阶段且本地不存在这些 testdata，应由 QA 在 `testdata/` 下补建后再执行。
4. 执行阶段必须：
   - 顺序运行测试，不并行执行多个用例
   - 同时检查 `maze.log`、`maze.py.log`
   - 对失败用例记录实际行为、复现步骤和可能原因
