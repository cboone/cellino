# 0016. Harnesses are executables behind their own build steps, never part of `zig build test`

**Status:** Accepted

## Context

Two of Savera's most important instruments run things rather than check them. The offline renderer drives the model with a score and a bellows curve and writes a WAV and state trajectories for the oracle ([ADR 0015](./0015-python-discovers-the-reed-and-freezes-as-oracle.md)). The offline host drives `process()` with synthetic events and judges hung notes, NaN, capacity refusals, `NOTE_END`, block splitting, tail and status transitions ([The verification program](../plans/2026-09-13-savera-build-plan.md#the-verification-program)).

Neither fits inside `zig build test`. A test binary must stay silent, because output from inside one interleaves with the test runner's stream and the build runner reads that as a failed step despite a zero exit code, so a harness running there cannot say what it was doing when it failed. The renderer writes files the unit suite has no business producing, and the host harness's CPU measurement is meaningless on a CI runner.

fosforo met the same problem with its GUI smoke harness and decided it in [its ADR 0013](https://github.com/cboone/fosforo/blob/1317e2b752f7f7d44db9bb7745200ca6e59dde83/docs/adr/0013-gui-smoke-harness-as-a-build-step.md). This ADR adopts that decision with Savera's harnesses in place of fosforo's.

## Decision

Harnesses are executables behind their own build steps: `zig build render` for `src/render.zig` and `zig build smoke` for `src/host_harness.zig`. None is ever wired into `zig build test`. Each reports through stderr and its own exit codes: 0 for a pass, 1 for a run that executed and failed, 2 for being invoked wrong. Where a harness's verdict is an absence, a judging script reads its output in an order that tells an absence apart from an instrument that did not run.

## Consequences

**`zig build` and `zig build test` stay what they are.** Harnesses are installed by build steps of their own, so the day-to-day loop builds only the plugin, and the unit suite stays hermetic.

**An absence is believed only after the instrument is shown to have run.** A judging script asserts, in order, that the harness passed, that its output exists and parses, and only then that the absence it is looking for holds, which is `plant-defects`' ordered-assertion rule. A comparison that finds no difference because the render was blank reads as a pass without that order, and NaN makes the same trap for every numeric bound, so the oracle rejects NaN explicitly.

**Each harness has a plant table, and the table regresses.** Phase 4's host-harness gate is a planted-defect matrix: a hung note, a NaN, a capacity refusal, a missed `NOTE_END`, per-segment rounding in block splitting, a wrong tail or status. Phase 2's oracle plants a detuned reed, a wrong aperture table and a swapped flow derivative. A plant expressible as data becomes a test that re-runs, rather than prose verified once.

**Some harness runs are by hand, and say why.** The CPU gate, `zig build smoke -Dreeds=16` in ReleaseFast, runs on the development machine with nothing else running, because a CI runner is not the target machine. The oracle and the smoke matrix run in CI.

**The Linux build path serves the harnesses.** `zig build render`, `zig build test` and `zig build smoke` build without macOS ([ADR 0001](./0001-macos-on-apple-silicon-only.md)), which is what lets the oracle row run beside the Ubuntu `python` job.

**What does not carry over from fosforo is its environmental risk.** fosforo's harness needs a Metal device and a window server, so part of it can fail for reasons unrelated to the code and is gated accordingly. Savera's harnesses need neither: the renderer and the offline host are pure computation over the model and the engine, so a red run is a finding about the code or about the harness, and neither is excused as flaky.
