## Test Cases Designed

### Test File(s)
- `test/py/test_maze_cli_byebye.py`: CLI regression tests for the new `--byebye` flag behavior.

### Cases
1. `test_byebye_prints_expected_output_and_exits_successfully`: verifies `./maze --byebye` exits with code 0, prints exactly `byebye`, and emits no stderr.
2. `test_byebye_does_not_fall_back_to_help_output`: verifies the new flag is handled directly instead of falling back to argparse help or the default no-pid flow.

## Notes for Developer
- Add `--byebye` as a first-class CLI flag in `maze` and short-circuit before normal standalone validation.
- Keep output exact and stable: a single line `byebye` with normal process exit.
