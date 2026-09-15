# 0015. Python discovers the single reed and is frozen as the oracle for that seam

**Status:** Accepted

## Context

Physics discovery is a loop of changing an equation and looking at what it does. In Zig, every "what does this look like" costs a recompile, and nothing in Zig plots. Python with Numba brings a scalar Newton loop within a small factor of compiled code and plots state variables directly, which is why the brainstorm proposed discovering the reed there and porting it.

The hazard of a two-language project is a parallel engine: two implementations of everything, drifting, with no rule for which one is right. The value of one is an oracle: a reference implementation that makes correctness measurable rather than ear-judged ([The harness and the oracle](../plans/2026-09-13-savera-build-plan.md#the-harness-and-the-oracle)).

## Decision

Python discovers the single reed, its aperture, its solver and its internal rate. Once the Zig port of `src/model/reed.zig`, `aperture.zig`, `flow.zig` and `solver.zig` agrees with it, the Python single-reed model is frozen as the oracle for that seam. Everything downstream of the reed seam, meaning the reservoir, the multi-reed coupling, the banks and the cabinet, is developed in Zig only, with an offline renderer as the shared instrument.

## Consequences

**The harness is scripts, not a package.** It lives in `harness/` as PEP 723 scripts with `#!/usr/bin/env -S uv run --script` shebangs and inline metadata pinning NumPy, SciPy, Numba and Matplotlib, following audio-tools, with `ruff.toml` at the root and a `python` CI job as fosforo has. Numba constrains the Python versions it supports, so the scripts pin one it supports when Phase 2 starts, the pin is recorded, and moving it is scheduled work.

**The renderer is the instrument downstream.** `zig build render` is an executable behind its own build step ([ADR 0016](./0016-harnesses-are-build-steps.md)) that takes a score and a bellows curve, renders a WAV, dumps the named state trajectories, and exits 0, 1 or 2. `harness/compare.py` and `harness/plot.py` consume its output. Its bordered-Newton reference mode is the multi-reed reference the explicit coupling is checked against ([ADR 0010](./0010-fixed-k-bracketed-newton-solver.md)). It is also how the owner hears the model before the plugin can play it.

**The oracle has two tiers, and both carry controls.**

- **Lockstep.** Python in fixed-K mode against Zig, both in `f64`, Zig in `.strict` float mode, with the same `K` and bracket rule, the same aperture table and interpolation, the same lagged chamber update, shared coefficients, and a defined initial state and onset seed. The tolerance is set from the measured agreement and recorded; bit-exact is the target and the actual figure is stated.
- **Perceptual.** Fundamental frequency in cents, partial levels in decibels, growth rate, the rising and falling thresholds, the onset pitch shift and the audible decay time, each with a tolerance chosen from the corpus's own spread.

**Every comparison rejects NaN explicitly**, because NaN compares false against every bound and a blank render passes a bare range check. Every gate has a row in a plant table, among them a detuned reed, a wrong aperture table and a swapped flow derivative, each confirmed to turn the comparison red before the gate counts as coverage. Analytic invariants that need no recording sit beside them: Fletcher's threshold as a function of volume and Q, playing frequency below the eigenfrequency, `τ = Q / (π f₀)` for the free ring-down, and Helmholtz choking.

**Shared constants cannot drift silently.** Constants restated in both languages are pinned by a Zig test that reads the Python source as text, the mechanism fosforo uses for its `measure-trace` script.

**The corpus has rules about what is committed.** Short CC0 reference cuts are committed to `fixtures/` so the oracle runs in CI; purchased audio, captures and renders live in the gitignored `verification/`. Figures digitised from papers are never committed; the numbers read off them are restated as constants in the harness beside the page they came from.

**Freezing is what makes the line hold.** After the freeze, a change to the Zig reed is a change the oracle judges, not a change the oracle follows. Moving `K` or the internal rate is an amendment to [ADR 0010](./0010-fixed-k-bracketed-newton-solver.md) or [ADR 0011](./0011-fixed-internal-sample-rate.md) and a regeneration of the oracle vectors.
