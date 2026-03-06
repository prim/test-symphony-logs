# Test Cases Designed

### Test File(s)
- common/mmap_owner_callsites_test.go: source-level regression tests that classify every MarkMmapOwner callsite and enforce precise-size expectations for known-safe and risky callers.

### Cases
1. TestMarkMmapOwnerAllKnownCallsitesAreClassified: verifies the complete set of production MarkMmapOwner callsites is enumerated and reviewed.
2. TestMarkMmapOwnerPreciseRangeCallersStayPrecise: verifies known precise-range callers continue to pass non-zero size arguments.
3. TestMarkMmapOwnerMetadataCallersDoNotClaimWholeMapping: verifies metadata pointers like tcmalloc span and jemalloc arena do not keep whole-mapping size=0 ownership calls.
4. TestMarkMmapOwnerWholeMappingCallersRemainExplicit: verifies intentional whole-mapping ownership callers still use explicit size=0 semantics.

## Notes for Developer
- These tests are intentionally source-based because the issue asks to audit every MarkMmapOwner callsite, not just runtime behavior.
- Current branch is expected to fail until risky metadata callsites are changed away from whole-mapping claims.
- go test ./common currently requires -vet=off in this worktree because unrelated vet warnings exist outside these new tests.
