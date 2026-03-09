## Test Cases Designed

### Test File(s)
- 无：本 issue 为纯文档分析任务，需求明确说明 QA 不需要参与测试，Dev 不需要编写功能代码。

### Cases
1. `doc_covers_resultitem_definition`: 验证新 dev log 明确指出 `common/result_items.go` 中 `ResultItem`、`PriorityQueue`、`ResultItemPool`、`GetResultItem`、`AddToResultItems`、`MergeResultItems`、`AddResult` 的职责与关系。
2. `doc_covers_upstream_usage_patterns`: 验证新 dev log 按使用模式归纳上层调用方，而不是只列单点引用，至少覆盖 `python/`、`nodejs/`、`mallocer/`、`cpp/`、`txos/`、`lua/`、`common/c.go` 等主要入口或模块。
3. `doc_explains_data_flow`: 验证新 dev log 说明调用链从“局部 resultItems 收集”到“MergeResultItems 合并到全局 ResultItems”的数据流。
4. `doc_notes_concurrency_and_pooling`: 验证新 dev log 说明对象池复用、`Mutex` 合并保护、PyMerge/HTTP/普通模式差异等关键设计点。
5. `doc_distinguishes_scope`: 验证新 dev log 明确本 issue 交付物是分析文档，不包含功能改动、测试代码或行为变更。

## Notes for Developer
- 该 issue 的核心产出是新的分析文档，而不是新增测试或实现。
- 文档应基于当前代码现状，优先总结“结构定义 + API 语义 + 典型调用模式 + 主要调用模块”。
- 若仓库中已有相关设计文档，可在新文档中补充“与现有文档的关系”，避免重复但要保证自洽。
