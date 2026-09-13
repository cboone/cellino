# Harmonium (peti) physical model: acoustics, synthesis engine, and phased build

## Scope

This issue covers **only** the acoustic modelling and sound synthesis side of Cellino: the reed model, the air supply, the instrument architecture, and the validation strategy. Plugin hosting, format, UI, parameter automation, and build infrastructure are deliberately out of scope here and belong in separate issues.

The target instrument is specifically the **Indian hand harmonium (peti)**: a small, pressure-blown, free-reed box where the player pumps a hinged bellows with the left hand and plays the keyboard with the right. Not the foot-pumped European harmonium, and not the suction-driven American reed organ — those differ in reed geometry, rank count, and air supply dynamics.

## Why physical modelling

The existing market is almost entirely sample-based (8Dio Harmonium, Syntheway, SwarPlug, various free Kontakt libraries). The notable exception is NUSofting's Peti SA, which uses a hybrid FM/physical-modelling engine. Sampling captures timbre but structurally cannot reproduce the three things that make a peti sound like a peti:

1. **Continuous bellows control of loudness**, independent of pitch.
2. **Shared-air-supply coupling** — every sounding reed draws on one small reservoir, so chords sag and thin.
3. **Nonlinear onset and decay** — the reed snaps into oscillation and rings down at its own rate, neither of which is an ADSR envelope.

A model also unlocks something no acoustic peti can do: **microtonality and continuous pitch glides**. Free reed pitch is set by reed length and is fixed at build time, which is the central limitation of the instrument in Hindustani practice. This is the explicit motivation behind Ninad Puranik's PhD work at McGill. It is a real differentiator and should be treated as a first-class goal, not a stretch feature.

## Core physics (the important part)

**A free reed has no pitch-setting resonator.** The reed swings freely through a close-fitting slot and never beats against anything. There is no bore, no standing wave, no delay line. **Digital waveguide methods are the wrong tool.** This is a lumped nonlinear ODE problem.

The reference formulation is the Millot–Baumann minimal model (arXiv:2401.01606). Reed tip displacement is treated as a single-degree-of-freedom oscillator standing in for the first transverse cantilever mode:

```
ζ̈ + (ω₀/Q)·ζ̇ + ω₀²·ζ = μ·Δp
```

closed by a quasi-stationary Bernoulli relation for the jet, `Δp = ½ρv_j²`, with output flow split into two contributions:

- a **pumped** component `S_r·ζ̇`, the reed physically displacing air;
- a **jet** component `α·S_u(ζ)·v_j` through the instantaneous opening.

Key consequences to get right:

- **`S_u(ζ)`, the "useful section", is where the timbre lives.** The reed's own motion is nearly sinusoidal; essentially all harmonic richness comes from the nonlinear geometry of the escape area as the tongue passes through the slot. Different published definitions of `S_u` produce audibly different attacks. This is the single highest-leverage modelling decision.
- The pumped flow is suspected to feed mostly low-frequency content, the jet flow the highs. Expose both separately in the prototype for inspection.
- This is a **blown-closed `(-,+)` configuration**: the reed plays slightly *below* its own natural frequency, and playing frequency is nearly independent of blowing pressure while amplitude varies strongly with it. That is exactly the peti's expressive character — loudness without pitch bend.
- Onset appears to be a **subcritical (inverse) bifurcation**: the note jumps into oscillation rather than growing smoothly, and once established it sustains at pressures below the starting threshold. This hysteresis is real physics, not an artifact, and it is what sampled instruments cannot fake.
- **Ring-down is Q-dependent, not envelope-shaped.** With Q ≈ 95, the amplitude time constant is roughly `Q/(π·f₀)` — about 70 ms at 440 Hz, nearly 280 ms at 110 Hz. Bass notes have visibly longer tails than treble.

Working overpressures in the literature sit in the range of a few hundred Pa up to about 1 kPa. Useful sanity check during calibration.

## Numerics

- Backward Euler for most of the system; bilinear transform with frequency pre-warping for the reed oscillator (following Gazengel's scheme as used by Millot).
- **Per-sample Newton–Raphson** on the mass-conservation residual for upstream overpressure, with a bisection fallback when the iteration oscillates.
- Precompute `S_u` and its derivative as a table.
- Run the model **oversampled** — 8× is a reasonable starting point given Q ≈ 95.
- The inner loop is scalar, sequential, and state-carrying. Iteration count is data-dependent and spikes at attacks. It cannot be vectorised.

## Beyond the single reed

The architecture points that matter more than the reed itself:

**Shared reservoir.** All active reeds draw from one pressure state. Model a two-stage pneumatic chain: pumping hand as a time-varying volume flow source → reservoir with compliance (spring-loaded, sets static working pressure) → short channel with inertance → reed chamber. This produces the swell-and-sag of real playing and the pressure drop when notes are added. **This is the thing that makes it sound like a harmonium rather than a stack of oscillators.**

**Reeds within a rank are coupled, not merely detuned.** On a double- or triple-reed peti the reeds for one key sit in a shared air channel and interact through local pressure. Expect frequency pulling and intermittent mode-locking — beating that wanders and occasionally catches, rather than a fixed beat rate. Do not implement this as static detune.

**Keys are valves, not velocity sensors.** Velocity should drive key/pallet noise and pallet opening rate, essentially nothing else. Loudness lives entirely in the bellows. Nearly every sampled harmonium gets this wrong.

**Bellows control mapping.** Expression pedal / breath / CC as primary. Also provide a synthesised pumping model (free-running or tempo-synced stroke generator) so the instrument is playable idiomatically from a plain keyboard.

**Cabinet as source-filter.** Follow Puranik & Scavone: treat the reed model as excitation and apply a filter representing the wooden enclosure, rather than modelling box acoustics. Their 2022 paper estimates the filter from a recorded note.

**Registration.** Peti-scale and small: two or three reed banks (16′ bass, 8′ male, 4′ female), octave coupler, and side drone knobs that sound continuously regardless of the keyboard. Drones are the constant CPU load and the clearest demonstration of reservoir coupling.

**Polyphony budget is generous.** One hand on the keys means ~4–6 simultaneous reeds worst case. Full per-sample Newton solving per reed is affordable; the hybrid shortcut is optional rather than necessary.

## Implementation approach

**Two-stage.** Physics discovery and validation in **Python with Numba** (`@njit` on the scalar Newton loop — NumPy gives nothing here, but Numba brings it within a small factor of C). Then **port to Zig** at the point where the multi-reed shared-reservoir architecture becomes the interesting problem, since that is a systems-design question rather than a numerical-discovery one.

The Python harness is **not** throwaway: it becomes the regression oracle. Render the same note from both implementations, compare spectra and attack envelopes, and correctness becomes measurable rather than ear-judged.

Rejected alternatives, for the record: Faust forbids unbounded loops (Newton would have to be a fixed unroll — correct for shipping, wrong for discovery) and has poor state introspection; straight-to-Zig has no plotting or REPL, so every "what does this look like?" costs a recompile; Octave matches the acoustics literature but is slow in loops with no Numba equivalent.

## Phasing

- **Phase 0 — harness.** Python + Numba scaffold, state-variable plotting (ζ, overpressure, jet velocity, both flow components separately), spectral comparison tooling, WAV rendering.
- **Phase 1 — single reed (initial release).** One reed, correct physics, constant pressure. Validate steady-state spectrum, the amplitude-vs-pressure curve, the subcritical onset, and Q-dependent ring-down. Ship this.
- **Phase 2 — bellows, transients, one rank.** Reservoir pneumatics, attack/release via pallet, key noise, velocity mapping, full keyboard range of one rank.
- **Phase 3 — full instrument.** Multiple ranks, coupler, drones, shared-reservoir coupling across all voices, channel coupling within a rank, cabinet filter.
- **Phase 4 — beyond the acoustic instrument.** Microtonality, continuous pitch glides, alternate temperaments (the peti is conventionally 12-TET, which is itself a known complaint in Hindustani practice).
- **Ongoing — hybrid fallback.** Evaluate modelled attack crossfading into a quasi-static additive sustain indexed by pressure. Puranik & Scavone's 2023 DAFx paper is a "physically inspired signal model" that does essentially this — a simplified, perceptually informed alternative to the full model. Worth implementing as a comparison branch and a CPU-constrained mode, not as the primary path.

## Validation without a physical instrument

**No access to a real peti**, so calibration leans on published parameter values plus recorded reference material. Strategy:

- Take reed dimensions, Q, and pressure ranges from the free-reed acoustics literature (Millot, Cottingham, Puranik & Scavone).
- Use commercial sample libraries and field/archival recordings as timbral ground truth: partial amplitude ratios, attack durations, beat rates between ranks.
- Compare spectra and attack envelopes quantitatively rather than by ear.
- **Known risk:** without direct measurement, absolute calibration of `S_u` geometry and reed damping is uncertain. Treat these as tunable and document the sensitivity. Revisit if instrument access becomes possible.

## References

**Primary modelling**

- Millot & Baumann, *A proposal for a minimal model of free reeds* — https://arxiv.org/abs/2401.01606
- Puranik, N. & Scavone, G. (2022), *Physical modelling synthesis of a harmonium*, Proc. Mtgs. Acoust. 49(1) 035015 — https://doi.org/10.1121/2.0001679
- Puranik, N. & Scavone, G. (2023), *Physically Inspired Signal Model for Harmonium Sound Synthesis*, DAFx-23, pp. 379–382 — https://www.dafx.de/paper-archive/2023/DAFx23_paper_47.pdf
- Puranik, N. & Scavone, G. (2023), *Clamped Bar Model for Free Reeds*, Forum Acusticum 2023
- CAML publications index — https://caml.music.mcgill.ca/doku.php?id=publications

**Interface / performance practice**

- Puranik, West, Wanderley & Scavone (2025), *Thoughts on Mapping and Interface Design of a Keyboard to Perform Continuous Pitch Ornamentations in Hindustani Music*, ICASSPW — https://caml.music.mcgill.ca/lib/exe/fetch.php?media=publications:puranik_icasspw_2025.pdf
- West, Puranik, Scavone & Wanderley (2025), *Towards the Continuous Harmonium: Replicating the Continuous Keyboard*, NIME — https://doi.org/10.5281/zenodo.15699652
- Background interview with Puranik on the research motivation — https://www.mcgill.ca/music/article/blog/conversation-ninad-puranik

**Code and architecture reference**

- Synthesis ToolKit in C++ (Cook & Scavone) — https://github.com/thestk/stk — useful for structure (`Instrmnt`, per-sample `tick()`, excitation/resonator/radiation wiring), **not** for algorithms. Its `ReedTable` models *beating* reeds and is the wrong physics. Note the authors' patent caveats in the class files.
- Faust physical modelling library — worth reading even though rejected as the prototype language.

**Prior art to study**

- NUSofting Peti SA — hybrid FM/physical-modelling peti and accordion, no samples. Closest existing competitor; worth buying to hear where hybrid FM succeeds and fails.
- Modartt Organteq — reference for how a modelled reed/pipe instrument exposes registration, tuning, and voicing as first-class parameters.
