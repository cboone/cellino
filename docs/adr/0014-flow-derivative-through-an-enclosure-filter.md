# 0014. The output is the derivative of total reed flow through an enclosure filter

**Status:** Accepted

## Context

The reed model produces flow. What reaches a listener is that flow radiated through a wooden box, and every reference recording in the corpus is heard through one. Puranik and Scavone treat the reed model as the excitation and the enclosure as a filter rather than modelling box acoustics: their all-pole filter, estimated by iterative adaptive inverse filtering from a recorded note, had 49 coefficients and eight or nine peaks below about 10 kHz, approximated by ten biquads. Their DAFx-23 paper states that "the filter has a much stronger influence on the synthesized sound timbre than the source" ([Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)).

Two modelling choices follow and both are easy to get wrong silently. Whether the filter is driven by flow or by its time derivative changes the spectral tilt by 6 dB per octave, and every oracle comparison depends on which. And the far field of a free reed is dipolar, which a single-channel filter does not represent.

## Decision

The output is the derivative of total reed flow, pumped plus jet, through an enclosure filter estimated from a recording. The jali cover open and closed are preset dimensions of that filter. The monopole approximation is deliberate, and the stereo output port carries it on both channels ([The cabinet](../plans/2026-09-13-savera-build-plan.md#the-cabinet)).

## Consequences

**The derivative is what Puranik convolves**, so it is what the published filter was estimated against, and the oracle's partial-level comparisons assume it. A swapped flow derivative is one of the oracle's planted defects.

**The cabinet filter arrives in the first playable build**, not a later phase, because the build plan's own finding is that no earlier build sounds like a harmonium without it. It ships as movable peaks with per-instrument presets, each fitted from the best-provenanced recording available for its target, with the jali open and closed as two presets. Peak controls are added only if the presets prove insufficient. The jali morph arrives with the full instrument in Phase 8.

**The monopole is a named approximation, and the dipole is deferred until it can be judged.** A dipole radiation model needs the filter and the recordings to exist first; until then there is nothing to measure it against.

**The port is stereo now because it cannot become stereo later.** Channel count is rescan-gated ([ADR 0023](./0023-one-note-port-and-one-stereo-output.md)), so two channels are declared from the first release, carrying the same monopole signal. The jali morph and any later dipole model are what would decorrelate them.

**This is not the hybrid engine.** DAFx-23's signal model is a wavetable read through ten biquads with an amplitude envelope: no modelled attack and no crossfade. It shares the filter idea and nothing else, and it stays deferred, reopened only if the CPU gate fails at the chosen polyphony.
