## Test Cases Designed

### Test File(s)
- `bird/test/issue-55.test.mjs`: Defines the expected public contract for the 3D flock simulation, gesture interaction, audio-reactive output, gameplay state transitions, and invalid-input handling.
- `bird/README.test-contract.md`: Documents the expected module path and exported API so the developer can implement against the tests.
- `bird/package.json`: Adds a runnable test command for the new bird feature test suite.

### Cases
1. `creates a runnable 3D flock scene with default birds`: verifies a default experience can be created and exposes birds with valid 3D position and velocity vectors.
2. `simulation step updates bird positions without producing invalid numbers`: verifies time stepping moves the flock while preserving finite numeric coordinates and velocities.
3. `valid hand gesture input changes flock interaction outcome`: verifies gesture-tracking input materially changes flock behavior and interaction state.
4. `missing gesture input degrades gracefully and still returns simulation output`: verifies null or missing gesture input does not crash the simulation and still yields flock/audio output.
5. `audio output changes when flock motion changes`: verifies the generated music state responds to different movement patterns instead of staying constant.
6. `pause, resume, and reset provide predictable gameplay state transitions`: verifies the interactive gameplay controls behave predictably and preserve/reset flock state correctly.
7. `rejects invalid configuration and step parameters with actionable errors`: verifies invalid bird counts, negative step deltas, and malformed gesture payloads are rejected with useful errors.

## Notes for Developer
- The tests assume the implementation entry point will be `bird/index.mjs` exporting `createBirdExperience(config?)`.
- The returned experience object is expected to provide `getState()`, `step(deltaMs, gesture?)`, `pause()`, `resume()`, and `reset()`.
- The tests intentionally fail now because the implementation does not yet exist; this is the expected TDD baseline for issue #55.
