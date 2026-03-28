## Test Cases Designed

### Test File(s)
- `tests/test_issue_171_todo_audit_regression.py`：校验 TODO 审计产物是否存在、是否覆盖全部来源、是否提供状态/价值/动作字段，以及是否按关键主题归类。

### Cases
1. `test_audit_report_exists_and_declares_structured_columns`：验证正式审计报告文件存在，并包含 `id/source/summary/module/status/value/evidence/action` 字段与稳定编号行。
2. `test_audit_report_covers_all_required_todo_sources`：验证审计报告覆盖根目录 TODO、文档 TODO、源码注释以及历史 dev-log 待办来源。
3. `test_audit_report_tracks_status_value_and_action_taxonomy`：验证审计报告声明 `DONE/PARTIAL/NOT_DONE/UNKNOWN`、`HIGH_VALUE/MEDIUM_VALUE/LOW_VALUE/OBSOLETE` 和后续动作分类。
4. `test_audit_report_groups_entries_by_key_focus_areas`：验证审计报告按 issue 指定重点方向归类，包括 Python、allocator、GDB/symbol/ctypes、printer、引用图、工具链、文档部署。

## Notes for Developer
- 这些测试把“TODO 审计结果”定义为一份正式文档契约：`dev-log/2026-03-28-todo-audit-report.md`。
- 当前仓库尚未提供该审计结果文档，因此测试按预期会失败；这是 TDD 期望行为。
- 若实现时希望采用 Markdown 表格以外的结构，至少需要保留稳定编号、字段声明、来源覆盖、状态/价值/动作分类与重点模块分组，否则测试不会通过。
