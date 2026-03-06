## Test Cases Designed

### Test File(s)
- `test/test_cli_byebye.py`: CLI regression tests for the new `--byebye` flag behavior.

### Cases
1. `test_byebye_prints_exact_message_and_exits_zero`: verifies `./maze --byebye` prints exactly `byebye` to stdout, writes nothing to stderr, and exits with code 0.
2. `test_byebye_does_not_require_pid_or_other_runtime_inputs`: verifies `--byebye` works as a standalone shortcut instead of falling through to default help / pid-required behavior.

## Notes for Developer
- Add `--byebye` as an early-exit CLI flag so it completes before normal standalone argument validation.
- Keep output stable and minimal because the test expects an exact stdout payload of `byebye\n`.
