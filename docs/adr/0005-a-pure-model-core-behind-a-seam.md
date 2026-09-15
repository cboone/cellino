# 0005. The model core is pure and names no CLAP type

**Status:** Accepted

## Context

Almost everything Savera computes is numerics. A reed's tip displacement, the jet through its useful section, the Newton iteration on the near-field pressure, the bank chamber's mass balance, the reservoir's spring law and the enclosure filter are all functions from numbers and state to numbers and state.

None of that needs a host, and all of it is where the defects will be. Several of the hazards appear only at specific inputs: a pressure difference crossing zero at onset, a chamber sagging toward threshold, a reed ringing down into subnormals. It is also the part that must agree with the frozen Python oracle ([ADR 0015](./0015-python-discovers-the-reed-and-freezes-as-oracle.md)), which is only meaningful if the Zig model can be driven with plain numbers and nothing else.

A physical model that can only be exercised by loading it into a digital audio workstation is a model tested by ear.

## Decision

Three layers, with the dependency arrow pointing one way only ([Source layout](../plans/2026-09-13-savera-build-plan.md#source-layout)):

- **`src/model/`** takes a sample rate, an oversampling factor and parameter values as plain arguments, and returns samples and named state. It may name a sample rate, because a DSP core cannot avoid one. It may not name a CLAP type, a sample offset or a note id.
- **`src/engine/`** knows about sample offsets, note ids, voices and events. It may not name a CLAP struct.
- **`src/clap/`** is the only place a CLAP type appears.

Enforce the innermost boundary mechanically, with a run-time source walk: `build.zig` hands a test the source root, the test walks `src/model/`, and it asserts that no file mentions `clap/c.zig` or an import path that resolves to the CLAP bindings.

## Consequences

**The enforcement is a run-time walk, not a comptime assertion, and the difference is deliberate.** springer's ADR 0005 states its seam as a comptime assertion that nothing in its core imports the CLAP bindings, on the precedent of fosforo's `src/gpu/iface.zig`. That comptime block pins function signatures, not import absence, so no "nothing under X imports Y" check exists in either sibling. Savera's check is the canary pattern from fosforo's `src/canary.zig` applied at run time, and because it walks the directory rather than an `@embedFile` list, a file added to `src/model/` is covered by construction.

**The check is proven by plant in Phase 2**: an import of `clap/c.zig` added to a model file must turn the test red. A seam check that never fails is indistinguishable from no seam check.

**The model is testable in full without a host**, and it is what the offline renderer drives: `zig build render` takes a score and a bellows curve and dumps a WAV and named state trajectories, which the oracle compares.

**The reed itself sits behind a second, inner seam** ([ADR 0009](./0009-single-mode-reed-behind-a-reed-seam.md)), so a modal reed can replace the single-mode oscillator without touching the flow, aperture, solver or air code.

The cost is that a few values are threaded as arguments that would otherwise be read from a parameter directly. A model function taking a sample rate and a pressure rather than a plugin handle is the rule working, not an inconvenience.
