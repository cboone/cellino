# 0010. Fixed-K bracketed Newton per reed on the near-field pressure

**Status:** Accepted

## Context

Millot's scheme integrates the air path by backward Euler and the reed by the bilinear transform with frequency pre-warping, following Gazengel, and solves one nonlinear equation per sample. His unknown is the reed-side pressure in a topology where the reed discharges to atmosphere, so one quantity is both the reed's downstream node and the pressure across it, and his residual is the upstream chamber's mass balance with one reed ([The solver](../plans/2026-09-13-savera-build-plan.md#the-solver)).

Savera's topology differs twice. The pallet sits downstream of the reed ([ADR 0012](./0012-reservoir-chambers-and-a-downstream-pallet.md)), so a near-field pressure exists between reed and pallet: the reed's load and jet are set by the chamber pressure minus that near-field pressure, and the pallet discharges it to the cabinet. And a bank has many reeds on one chamber. Neither is in any published solver.

Two further constraints bind the choice. Puranik's lumped harmonium adaptation needed unrealistic parameters to stay stable and per-note retuning of the non-playing parameters, so solver stability under parameter sweeps is the project's primary physics risk. And [ADR 0007](./0007-no-allocation-on-the-audio-thread.md) forbids unbounded work on the audio thread, which rules out iterating to a tolerance.

## Decision

Per internal sample and per reed, solve one nonlinear equation in that reed's near-field pressure, whose residual is flow continuity at the near-field node: reed flow in, pumped plus jet through the chamber-to-near-field difference with the chamber pressure held at its previous-sample value, equals pallet flow out through the near-field-to-cabinet difference, with the near-field pipe's inertance carried as state between them. Solve it with bracketed Newton at a fixed iteration count `K`. Update the bank chamber and the reservoir explicitly after the per-reed solves. `K`, the jet crossover and the worst case are measured in the harness, and an overrun counter is logged from the main thread.

## Consequences

**The bound is structural.** A Newton step that leaves the bracket, the physical range between the cabinet pressure and the chamber pressure, is replaced by a bisection step. An iteration that reaches `K` carries its last iterate and increments the overrun counter, a relaxed atomic drained on the main thread. The work per sample is bounded by construction rather than by hope.

**`K` is open until Phase 2 measures it.** The working assumption for budgets is 4. The harness runs two modes, converge-to-tolerance for discovery and the identical fixed-K mode, and reports the fixed-K error per event class (steady state, attack, pressure step, release), so the choice of `K` is a measured trade. The value is recorded as an amendment here; changing it after that is a further amendment and a regeneration of the oracle vectors, which CI turns red until it happens.

**The residual carries two square roots**, one on each side of the node. With the pallet fully open the near-field pressure sits at the cabinet's and the equation reduces to Millot's; partly closed, it rises, which is where the loudness reduction and the acoustic-load pitch pull come from ([ADR 0013](./0013-keys-are-continuous-valves.md)). Whether the residual stays single-rooted over the bracket is a Phase 2 measurement.

**The Bernoulli relation is regularised.** `v_j = sqrt(2Δp/ρ)` has infinite slope at zero pressure difference, which Newton crosses at every onset and every sag to threshold. On `[0, Δp_c]` the jet is a cubic Hermite with zero value and finite slope at the origin and matched value and slope at the crossover; above it the relation is odd-symmetric, because the pressure difference reverses. The alternative, matching value only and accepting a slope jump, is measured against it. The crossover is recorded here as an amendment.

**The explicit coupling has a stability condition, and it is checked.** The coupling is stable while the chamber's time constant `τ_c = C / Σⱼ (duⱼ / dp)` stays long against the internal sample period, with `C = V1 / (ρ₀ c₀²)`. The margin shrinks with polyphony and near the regularised branch. The render harness carries a bordered-Newton reference mode over all of a bank's reeds, and Phase 5 checks the explicit scheme against it at the calibrated values and at the bank's worst-case reed count before the engine commits to it. That result is recorded as an amendment here.

**Explicit coupling is what permits lanes.** Reeds are independent within a sample, so `@Vector` lanes can vectorise the arithmetic across them. Neither Zig nor Apple's NEON has a gather, so table lookups stay scalar loads into an array. Phase 2 measures the lane arm against the scalar baseline on sixteen independent reeds, so the figure is known before `K` and the rate are frozen.

**The numerics are fixed so that the oracle is meaningful.** Solver state and tables are `f64`, output is `f32`, and the solver module runs in `.strict` float mode so the residual is identical in every optimize mode. The harness single-reed model runs the same lagged chamber update the engine does, so the lockstep tier compares like with like, and Millot's simultaneous scheme is run once to bound the lag's effect, with the figure recorded.
