## Test Cases Designed

### Test File(s)
- `common/results/typeid_test.go`: verifies the planned TypeID bitfield API can construct, decode, and stringify debug-friendly IDs.
- `common/results/strategy_test.go`: verifies ResultItem stats, simple aggregation, and Top-N aggregation semantics.
- `common/results/collector_test.go`: verifies the planned Collector can aggregate, merge staged/local results, finalize statistics, and reset state.

### Cases
1. `TestMakeTypeIDRoundTrip`: verifies `MakeTypeID` preserves module, subtype, and sequence fields.
2. `TestTypeIDStringContainsDebugFields`: verifies `TypeID.String()` includes debuggable module/subtype/sequence content.
3. `TestResultItemStatsComputesAverage`: verifies `ResultItem.Stats()` returns correct count, total size, and average size.
4. `TestSimpleStrategyAggregatesCountsAndBytes`: verifies `SimpleStrategy` accumulates object count and total bytes.
5. `TestTopNStrategyKeepsLargestObjectsInDescendingOrder`: verifies `TopNStrategy` keeps only the largest N objects and finalizes them in descending order.
6. `TestCollectorAddGetAndGetAll`: verifies a Collector can add objects, fetch one aggregated item, and return a snapshot of all items.
7. `TestCollectorMergeFinalizeAndReset`: verifies Collector merge/finalize/reset behavior for staged local results.

## Notes for Developer
- These tests intentionally target the API sketched in `dev-log/2026-03-08-results-redesign-clean.md`.
- The new package is expected at `common/results` with concrete types such as `TypeID`, `ResultItem`, `ObjectInfo`, `Collector`, `TypeRegistry`, `SimpleStrategy`, and `TopNStrategy`.
- Tests currently fail because the refactor has not been implemented yet; this is intentional TDD coverage for issue #51.
