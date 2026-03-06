## Test Cases Designed

### Test File(s)
- `test_byebye.py`: CLI regression tests for the new `--byebye` option on the top-level `maze` launcher.

### Cases
1. `test_byebye_prints_expected_message_and_exits_successfully`: verifies `./maze --byebye` exits with code 0, prints exactly `byebye`, and does not emit stderr output.
2. `test_byebye_does_not_fall_back_to_default_help_output`: verifies the new option is handled explicitly instead of falling through to the default help/usage output path.

## Notes for Developer
- The current behavior treats `--byebye` as an unknown argument and falls through to the no-PID help path; these tests intentionally fail until the option is implemented.
- Keep the implementation in the Python launcher (`maze`) so the command can exit before any build, GDB, or normal analysis flow is triggered.
