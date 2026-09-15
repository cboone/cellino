# 0012. A reservoir feeds sealed bank chambers, with the pallet downstream of the reed

**Status:** Accepted

## Context

The shared air supply is what makes a harmonium sound like one rather than like a stack of oscillators: chords sag and thin, and a single note holds for ten to twenty seconds while a chord holds for less. The model has to produce that from mass conservation, because no published model has more than one reed drawing on one volume.

The brainstorm proposed the reservoir as air compressibility. That cannot hold a note: an 18 litre box has an adiabatic compliance of `V / (ρ₀ c₀²)`, about `1.3e-7 m³/Pa`, so it stores about 130 millilitres at 1 kPa, roughly one second of one reed's draw and an order of magnitude short of a ten-second hold ([The air path](../plans/2026-09-13-savera-build-plan.md#the-air-path)). Every repair source describes a spring-loaded internal bellows instead.

Two further facts about the instrument's construction shape the topology ([Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)). Reeds are screwed to the underside of the reed board and pressurised from below, and the key's pallet seals the hole above the reed, so the pallet is downstream of the reed, not upstream. And a peti's banks sit in chambers sealed off from one another, an octave apart; unison reed pairs exist only on custom Male-Celeste instruments and the 22-shruti harmonium, and no harmonium literature documents in-rank coupling or mode-locking.

## Decision

The air path, upstream to downstream, is hand, external bellows, one-way flap, a spring-loaded reservoir with a leak, a stop orifice per bank, a sealed bank chamber per bank, the reed, a near-field pipe per key, a pallet orifice per key whose area follows key depth, and the cabinet. In-rank coupling terms are not added.

## Consequences

**Reeds are coupled through the chamber and the reservoir, and nowhere else.** Every reed in a bank sees the same chamber pressure and every bank sees the same reservoir, which is what produces sag. The shared chamber's ripple is about plus or minus 20 Pa on 450 Pa in Puranik's adapted model. No mode-locking behaviour is a requirement or a gate.

**In-rank coupling is a recorded refusal.** A Male-Celeste bank with a detuned unison pair is its eventual home if one is ever wanted, with the accordion celeste range of 0.5 to 15 Hz as the target; it is deferred, not forgotten.

**Chamber resonance is produced, not tuned away.** The chamber compliance and the pipe inertance are both state, so the model makes whatever resonance the calibrated chamber and its open pipes have. Cottingham reports a reed cell resonance raising the threshold by an order of magnitude and pulling a 344 Hz reed to 312 Hz, and the oracle's Helmholtz choking invariant checks the behaviour is reproduced.

**Two questions are left to Phase 2, and the harness implements both answers.** Whether the near-field pipe belongs upstream of the reed, as in Millot's topology and Puranik's reuse of it, or downstream, where the target instrument's geometry puts it; and whether the chamber volume `V1` is the single-reed figure or scales with the reeds in the bank. The findings are recorded as amendments here.

**The starting values are placeholders by their own source's admission.** FA2023 gives `V1 = 1.3e-3 m³`, `L2 = 50 mm`, `S2 = 6.4e-5 m²` and `ρ₀ = 1.1769 kg/m³`, and states that `V1`, `S2` and `L2` are arbitrary in that model; the calibration pass sets them.

**The calibration targets carry their provenance.** Reed-chamber pressure between 200 and 1000 Pa with sharp rises on strokes and exponential decay between them (Puranik and Scavone, POMA 2022, Fig. 5); five to eight seconds of leak-down with every stop closed and ten to twenty seconds for a single held note (repair notes); a soundbox of about 18 litres and an external bellows of about 2.75 litres (one weak source, to be re-derived from the model's hold times). Millot's per-reed flow of zero to 160 millilitres per second is a harmonica simulation and so a plausibility bound, never a gate.

**The sag is an untested inference, and that is a standing risk** until Phase 5 measures sag, leak-down and single-note hold against those targets.
