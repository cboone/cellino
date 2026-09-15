# 0019. Savera models a pressure-blown Indian hand harmonium and nothing else

**Status:** Accepted

## Context

The target is specific: a Kolkata-style three-bank peti with Palitana reeds, a small pressure-blown free-reed box whose player pumps a bellows with one hand and plays with the other ([What we are building](../plans/2026-09-13-savera-build-plan.md#what-we-are-building)). Neighbouring instruments and techniques are close enough to tempt scope creep, and each one would change what the plugin is:

- **Sample playback.** The existing market is almost entirely sample-based. Sampling captures timbre but structurally cannot reproduce continuous bellows loudness independent of pitch, shared-supply sag, or onset and ring-down that follow the physics rather than an envelope.
- **Waveguides.** A free reed has no pitch-setting resonator, no bore and no standing wave; a delay-line model is the wrong physics, and the Synthesis ToolKit's reed table models beating reeds.
- **The European harmonium.** Foot-pumped, with different reed geometry, rank counts and air supply dynamics.
- **The American reed organ.** Suction-driven rather than pressure-blown, which reverses the air path.

## Decision

Savera models a pressure-blown Indian hand harmonium and nothing else. No sample playback of any kind, no waveguide, no European harmonium and no American reed organ. These refusals are recorded on purpose.

## Consequences

**A refusal is a decision, not a gap.** A review that finds "no sample layer" or "no organ preset" has found this ADR. A genuine new reason to revisit one is a superseding ADR, not a feature request.

**Variation happens inside the instrument.** The full instrument's Delhi and Lahore presets differ from the Kolkata target in reed stiffness and damping, cabinet, coupler default and drone count, which are all peti parameters, rather than in registration alone. Past the acoustic instrument, the reed and reservoir parameters unlock beyond their real ranges ([ADR 0008](./0008-parameters-identified-by-stable-clap-id.md)), which is modulation of the model, not a second model.

**Recordings of other instruments are references, never sources.** NUSofting's Peti SA, a hybrid FM design with no Apple Silicon build, runs under Rosetta only as a control-surface parity check, never as ground truth. Organteq is cited for registration and tuning interface only, since its voicing is recomputed offline and is the opposite of modulatable.

**What is deferred is not refused.** In-rank coupling with a Male-Celeste bank ([ADR 0012](./0012-reservoir-chambers-and-a-downstream-pallet.md)), the modal and clamped-bar reeds, a dipole radiation model, a keyboard beyond 42 keys, and the hybrid signal-model engine are all peti modelling choices deferred behind measurements ([Explicitly deferred](../plans/2026-09-13-savera-build-plan.md#explicitly-deferred)). VST3, AUv3, AAX and standalone builds are a later clap-wrapper toggle, not a change of instrument.
