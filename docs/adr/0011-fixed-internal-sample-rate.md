# 0011. The model runs at a fixed internal rate, resampled to the host rate

**Status:** Accepted

## Context

Backward Euler's first-order damping makes the threshold pressure, the growth rate at onset and the pressure-to-pitch coefficient depend on the sample rate the model runs at. A design that runs the model at the host rate times an oversampling factor would therefore sound different in a 44.1 kHz session and a 96 kHz one, and would defeat the oracle, whose lockstep tier needs the Zig model and the Python model to integrate the same equations at the same rate ([ADR 0015](./0015-python-discovers-the-reed-and-freezes-as-oracle.md)).

The brainstorm's 8x oversampling has no source. The only published rate in this model family is 4 times 44.1 kHz, for the clamped-bar model in FA2023 ([Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)).

## Decision

The model runs at one fixed internal sample rate, chosen by measurement in Phase 2 and then frozen as a compile-time constant. A rational polyphase resampler carries it to the host rate. The resampler's latency is reported through `clap.latency`, and the remaining ring-down through `clap.tail` ([The internal rate and the resampler](../plans/2026-09-13-savera-build-plan.md#the-internal-rate-and-the-resampler)).

## Consequences

**The rate is open until Phase 2 measures it.** The study measures the convergence of the threshold pressure, the growth rate and the pressure-to-pitch coefficient, and the folded-back aliasing, at two, four, eight and sixteen times 48 kHz. The working assumption for budgets is 192 kHz. The chosen rate is recorded as an amendment here; changing it afterwards is a further amendment and a regeneration of the oracle vectors.

**Host rates are a tabulated set, and anything else is refused.** Ratios for the 44.1 and 48 kHz families are tabulated at `activate` into storage sized for the largest, and `activate` refuses a host rate outside that set with a runtime `if`.

**Block splitting has to respect a ratio that is not an integer.** Events arrive with frame offsets at the host rate, and the block is split at every event offset. Host frame `k` maps to internal position `k R`, with `R` = 640/147 at 44.1 kHz against 192 kHz, so a segment's internal sample count is the difference of the floors at its two ends and varies by one from segment to segment. The resampler's phase accumulator and polyphase history persist across segments and blocks and are reset only by `reset()` and `activate()`. A `zig build smoke` case drives one block at 44.1 kHz with an event on every frame and asserts both that the block emits exactly the host frame count and that its output equals the same events delivered unsplit, which is the planted-defect arm for per-segment rounding.

**The reported latency is measured, not asserted.** Phase 4's gate requires it to match a measured impulse within one internal sample.

**Continuous inputs are smoothed at the internal rate**, with a time constant that is itself a parameter, because a fixed smoothing ramp is what makes digital accordions miss the first strokes of a bellows shake.

**The tail reflects the physics, including states that never end.** `clap.tail` reports the longest remaining ring-down in host samples, from the slowest sounding reed's measured decay plus the cabinet filter's, and a value at or above `INT32_MAX` whenever a drone stop is open or the pump generator is running, so a host does not truncate a bounce. That is CLAP's own sentinel: `clap/ext/tail.h` in CLAP 1.2.10 states that "any value greater or equal to INT32_MAX implies infinite tail", and that version defines no separate named constant for it. `process` returns `CLAP_PROCESS_CONTINUE` while any reed, drone or reservoir state is non-quiet, `CLAP_PROCESS_TAIL` while only ring-down remains, and never `CLAP_PROCESS_SLEEP` with a drone open.

**The Audio Unit reports no tail regardless, and that is a wrapper deficit.** clap-wrapper v0.16.0 hard-codes `SupportsTail()` to false while implementing `GetTailTime()` ([ADR 0003](./0003-author-clap-project-outward.md)). The by-hand Logic bounce step records what a released bass note and a held drone do under it.

The cost is a resampler in every signal path and the latency it adds, which is small against the alternative of an instrument whose onset depends on the session's sample rate.
