# 0007. Nothing on the audio thread allocates, locks, or makes a syscall

**Status:** Accepted

## Context

Memory safety is not real-time safety. A heap allocation, a mutex acquisition or a file read compiles, passes every test, and still produces an audible dropout when the allocator takes a slow path or the lock is contended. The failure is intermittent, load-dependent, and does not reproduce under a debugger.

Savera puts more work on that thread than either sibling. Every internal sample runs a bounded Newton solve per sounding reed, at an internal rate several times the host's ([ADR 0010](./0010-fixed-k-bracketed-newton-solver.md), [ADR 0011](./0011-fixed-internal-sample-rate.md)), and state that must survive across blocks includes the reed slots, the resampler's history, the reservoir and the chamber pressures. A dropout here is not a slow frame; it is a click in someone's recording.

## Decision

Nothing reachable from `process()` allocates, takes a lock, or makes a syscall. Every capacity is derived in one place, with the derivation written beside the number. Cross-thread state is single-writer relaxed atomics, drained on the main thread.

## Consequences

**Capacities are derived, and the derivation is the documentation.** The reed-slot capacity of 32, the key cap of 8 and the per-bank table of 54 reeds live in one table with their arithmetic beside them ([Voices, capacity and the CPU budget](../plans/2026-09-13-savera-build-plan.md#voices-capacity-and-the-cpu-budget)), so a later change to the fan-out has a visible consequence. A demand above capacity is a defined behaviour, the written steal policy, not an overflow; a note outside the keyboard is refused and counted.

**Buffers are sized once, at `activate`.** Every internal-rate buffer is sized from the host's maximum frame count times the internal-to-host ratio times the reed capacity, from one fixed-size arena taken once. `process` refuses a block larger than the declared maximum with a runtime `if` and counts it, and a test asserts the arena's end index returns to zero on every block, as fosforo's does.

**Cross-thread data has one writer and is drained on the main thread.** The solver's overrun counter is a relaxed atomic the audio thread increments and the main thread drains in the parameter flush. The parameter store keeps a main-thread copy that `state.save` reads while the audio thread's copy is updated from events, so the two never race. Diagnostics from the audio path set state that the main thread reports through `clap.log`; nothing on the audio path writes anywhere.

**Savera starts no thread of its own, and a source canary says so.** Because every cross-thread datum is single-writer and drained on the main thread, no ordering between two writers exists to verify, so no Thread Sanitizer harness is built. A canary asserts that nothing under `src/` spawns a thread, which keeps that argument true rather than assumed.

**Subnormals are handled in the register, not in the float mode.** A reed rings to minus 60 dB over seconds and reaches subnormals on every release. Flush-to-zero is set through bit 24 of the ARM64 floating-point control register, saved and restored around `process()`, because `@setFloatMode` does not touch that register. A runtime check the optimizer cannot fold confirms it, verified by plant.

**Trust boundaries refuse with a runtime `if`, never an `assert`.** The shipped CLAP is ReleaseFast, where `std.debug.assert` is gone, so the unit suite runs in `test`, `test-safe` and `test-release`. A solver whose convergence were guarded by an assert would hide its failure in exactly the build that ships.

[ADR 0002](./0002-zig-pinned-to-0-16-0.md) is what makes this checkable: nothing on the audio path is handed an allocator, so "does this allocate" is answered by reading the call graph. The cost is that fixed capacities are real limits. A thirty-third simultaneous reed is stolen, not served, and that is the correct trade for an instrument whose worst realistic demand has been derived.
