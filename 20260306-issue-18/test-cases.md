## Test Cases Designed

### Test File(s)
- `test/py/test_byebye_cli.py`: CLI regression tests for the new `--byebye` flag behavior.

### Cases
1. `test_byebye_prints_expected_output_and_exits_successfully`: verifies `./maze --byebye` exits with code 0 and prints exactly `byebye` plus trailing newline.
2. `test_byebye_does_not_fall_back_to_help_or_argument_errors`: verifies `./maze --byebye` does not emit help text, argument dump output, or stderr noise.

## Notes for Developer
- The new flag should short-circuit before normal CLI validation/build/analyze flows.
- The observable contract is strict: stdout should be exactly `byebye\n`, stderr should stay empty, and exit code should be 0.
