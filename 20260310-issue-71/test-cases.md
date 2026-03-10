## Test Cases Designed

### Test File(s)
- `tests/test_maze_loop_integration.py`: Covers CLI help, main dispatch, loop/kill-loop helper behavior, psutil error handling, and removal of legacy scripts for issue #71.

### Cases
1. `test_help_includes_loop_options_and_examples`: verifies `./maze --help` documents `--loop`, `--kill-loop`, and their usage examples.
2. `test_main_dispatches_loop_mode_without_requiring_pid`: verifies `main()` dispatches `--loop` directly instead of falling back to standalone PID validation.
3. `test_main_dispatches_kill_loop_mode_without_requiring_pid`: verifies `main()` dispatches `--kill-loop` directly instead of requiring `--pid`.
4. `test_loop_and_kill_loop_helpers_are_defined`: verifies the integrated helper functions exist in the `maze` entry script.
5. `test_do_loop_repeats_build_run_and_exits_gracefully_on_keyboard_interrupt`: verifies loop mode runs the expected build+run command and exits cleanly on Ctrl+C.
6. `test_kill_loop_reports_missing_psutil_helpfully`: verifies kill-loop mode emits a friendly installation hint when `psutil` is unavailable.
7. `test_do_kill_loop_terminates_matching_loop_process_and_children`: verifies kill-loop mode finds a `maze --loop` process, kills child processes first, then the parent.
8. `test_legacy_loop_scripts_are_removed`: verifies `loop.py` and `kill_loop.py` are removed after integration.

## Notes for Developer
- Tests intentionally fail until `maze` adds `--loop` / `--kill-loop`, defines `do_loop()` / `do_kill_loop()`, updates help text, and deletes the legacy scripts.
- `do_kill_loop()` is expected to detect loop processes by checking `cmdline` for `--loop` and avoid killing its own PID.
- `do_loop()` should execute `./maze --build --run --mode postman --tag dev` repeatedly and handle `KeyboardInterrupt` without a traceback.
