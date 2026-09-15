# 0009. The reed is a single-mode oscillator behind a reed seam

**Status:** Accepted

## Context

A free reed has no pitch-setting resonator. The tongue swings through a close-fitting slot without beating against anything, so there is no bore, no standing wave and no delay line; the reed is a lumped nonlinear oscillator driven by the pressure difference across it, and the timbre lives largely in the geometry of the escape area as the tongue passes through the slot.

The reference formulation is Millot and Baumann's minimal model, published in Acta Acustica united with Acustica 93 (2007), pages 122 to 144, and posted to arXiv in 2024 as 2401.01606. Its parameters describe a harmonica reed, and the paper states it was never confronted with experiment ([Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)). Exactly one harmonium reed has published geometry, material and damping: the B5 reed of Puranik and Scavone's Forum Acusticum 2023 clamped-bar paper, 16 mm by 2 mm by 0.4 mm with a 0.1 mm tip offset, Young's modulus 125 GPa and density 8490 kg/m³, measured by laser Doppler vibrometry at two pressures.

Two richer reeds exist in the literature. A two- or three-mode modal reed projects the pressure load onto the first transverse modes of a clamped-free beam. The clamped bar of FA2023 resolves 40 grid sections, at roughly forty times the reed-update cost before the flow solve. Neither is justified before the simplest reed has been measured against the corpus.

## Decision

The reed is a single-degree-of-freedom oscillator standing in for the first transverse cantilever mode, `ζ̈ + (ω₀ / Q) ζ̇ + ω₀² ζ = μ Δp`, blown closed in Fletcher's `(−,+)` configuration, with Millot's pumped flow `S_r ζ̇` and jet flow `α S_u(h) v_j`. It sits behind a reed seam that exports what the solver needs and is sized for a modal reed ([The reed](../plans/2026-09-13-savera-build-plan.md#the-reed)).

## Consequences

**The seam is what makes the reed replaceable.** `src/model/reed.zig` exposes a state, a pressure load in, a tip position and velocity out, and the two sensitivities the solver's Jacobian needs for the current step, `dζ/dΔp` and `dζ̇/dΔp`. For the bilinear single-mode reed those are the two bilinear coefficients; for a modal reed they are mode-shape-weighted sums of per-mode coefficients; for the clamped bar they come from its implicit step. A modal reed or the clamped bar therefore replaces this one without touching the flow, aperture, solver or air code ([ADR 0010](./0010-fixed-k-bracketed-newton-solver.md)).

**Its limits are named deficits, not hidden ones.** The single-mode reed is valid to about 1 kPa: FA2023's vibrometry shows a sinusoidal reed at 0.2 kPa and a harmonic-rich one at 1.4 to 1.6 kPa, and every measured attack shows a second transverse and a first torsional mode that one mode cannot produce. Both are recorded as deficits of this decision, and the modal reed is deferred until the corpus shows they matter.

**The aperture is pluggable and its baseline is stated.** The baseline useful section is Millot's own definition, his equation 9, with the B5 geometry. It also needs a side clearance that no harmonium source publishes, so the clearance is fitted in Phase 2, bracketed by Millot's harmonica 50 µm and the Euphonics harmonica's 0.2 mm, with its sensitivity reported. Puranik's simplified aperture and Tarnopolsky's form are implemented alongside it in the harness and compared. Tables hold geometry only, `S_u(h)` and `dS_u/dh`, with the chain-rule factor from the bilinear step computed separately, so a change of rate or a modulation of `ω₀` or `Q` never leaves a stale table.

**Damping starts as a bracket, not a value.** Millot's harmonica Q of 95 and the Q of about 650 that FA2023's beam damping coefficient implies for the B5 reed differ by a factor of seven in ring-down time: by `τ = Q / (π f₀)`, 31 ms against 209 ms at 990 Hz. Phase 2 carries both, and per-reed Q is fitted from the corpus ring-downs.

**Pressure lowers the pitch, and only the sign is known.** Measurements show a small, perceptible downward drift with pressure, and the single-mode model produces that sign; its coefficient in a harmonium configuration is unmeasured. The sign is an oracle target. The coefficient is measured in Phase 2 and recorded as an amendment here, the Pressure-to-Pitch Scale parameter absorbs whatever the calibration finds short, and if the shortfall is large an aerodynamic stiffness term is the candidate, recorded as a further amendment.

**Every other reed in a bank is generated, not hand-set.** From the B5 anchor, the cantilever family `f ∝ t / L²` scales thickness and length with width and material held fixed ([ADR 0018](./0018-tuning-tables-and-a-reed-family.md)); per-reed Q and a tuning offset are the only hand-set quantities. The table is comptime, 54 reeds per bank. Real bass reeds are weighted rather than scaled, and that deviation is a recorded deficit.
