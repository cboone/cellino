# savera

## Overview

A modulatable synthesizer built on a physical model of the Indian hand harmonium (peti), for macOS (CLAP and AUv2).

Savera models a Kolkata-style peti: a small pressure-blown free-reed box whose player pumps a bellows with one hand and plays the keyboard with the other. It is authored once as a CLAP instrument in Zig and projected outward by clap-wrapper to an Audio Unit, so it runs in Logic Pro as a software instrument and in REAPER and Bitwig as a CLAP. Two goals pull against each other: realism, with the reed, the air supply and the wooden box modelled from the free-reed acoustics literature and calibrated against recordings; and modulation, with every physical quantity the engine reads per sample exposed as a modulatable parameter. Realism is the foundation; modulation is the point.

The name is Hindi and Urdu for "dawn", said sa-VEH-ra. The product and display name are both `Savera`, with no diacritic, and the repository and file names are ASCII `savera`.

## Current state

Phase 0 of eleven, the repository foundation: agent config, lint and secret-scanning CI, the twenty-three ADRs, and the notes. **Nothing builds yet, by design.** There is no `build.zig`, no `build.zig.zon` and no `src/`; they arrive in Phase 1 with `ci.yml`, which reads its toolchain from `build.zig.zon`.

The sequencing, the phase gates, the findings and the verification program live in [the build plan](docs/plans/2026-09-13-savera-build-plan.md). **It is permanent and never leaves `docs/plans/`**; per-phase plans are separate dated files in `docs/plans/todo/` that move to `docs/plans/done/` when their pull request merges. What has landed is recorded in the build plan's phase outcomes subsections, not here.

## Non-negotiables

These are settled decisions recorded in [`docs/adr/`](docs/adr/README.md). Do not relitigate them in code review; supersede them with an amendment or a new ADR instead.

- **macOS on Apple Silicon only.** Not a portability oversight ([ADR 0001](docs/adr/0001-macos-on-apple-silicon-only.md)).
- **Zig is pinned to 0.16.0.** `build.zig.zon` is the single source of truth and CI reads it; compiler bumps are scheduled work ([ADR 0002](docs/adr/0002-zig-pinned-to-0-16-0.md)).
- **Authored once, as a CLAP instrument.** clap-wrapper projects it to an AUv2 `aumu`, with the type passed explicitly and read back from the built plist as the control ([ADR 0003](docs/adr/0003-author-clap-project-outward.md)).
- **CLAP bindings come from translate-c over a `zig cc -E -P` step**, with comptime layout assertions over every struct crossing the ABI ([ADR 0004](docs/adr/0004-clap-bindings-via-translate-c.md)).
- **Nothing under `src/model/` names a CLAP type, a sample offset or a note id.** A run-time walk of that directory enforces it ([ADR 0005](docs/adr/0005-a-pure-model-core-behind-a-seam.md)).
- **No WebView UI** ([ADR 0006](docs/adr/0006-reject-webview-ui.md)).
- **Nothing reachable from the audio thread may allocate, lock, or make a syscall.** Capacities are derived in one place; cross-thread state is single-writer relaxed atomics drained on the main thread ([ADR 0007](docs/adr/0007-no-allocation-on-the-audio-thread.md)).
- **Parameters are identified by stable `clap_id`.** Every rescan-gated flag and range is declared before the first release ([ADR 0008](docs/adr/0008-parameters-identified-by-stable-clap-id.md)).
- **The reed is a single-mode oscillator behind a reed seam** that exports the solver's sensitivities and is sized for a modal reed ([ADR 0009](docs/adr/0009-single-mode-reed-behind-a-reed-seam.md)).
- **The solver is fixed-K bracketed Newton per reed on the near-field pressure**, with the chamber and reservoir updated explicitly afterwards ([ADR 0010](docs/adr/0010-fixed-k-bracketed-newton-solver.md)).
- **The model runs at a fixed internal rate, resampled to the host rate** ([ADR 0011](docs/adr/0011-fixed-internal-sample-rate.md)).
- **A spring-loaded reservoir feeds sealed bank chambers, and the pallet is downstream of the reed.** No in-rank coupling terms ([ADR 0012](docs/adr/0012-reservoir-chambers-and-a-downstream-pallet.md)).
- **Keys are continuous valves.** Velocity drives only the pallet's opening rate and the key noise ([ADR 0013](docs/adr/0013-keys-are-continuous-valves.md)).
- **The output is the derivative of total reed flow through an enclosure filter**, monopole on both stereo channels ([ADR 0014](docs/adr/0014-flow-derivative-through-an-enclosure-filter.md)).
- **Python discovers the single reed and is frozen as its oracle.** Everything downstream is Zig only ([ADR 0015](docs/adr/0015-python-discovers-the-reed-and-freezes-as-oracle.md)).
- **Harnesses are executables behind their own build steps, never part of `zig build test`** ([ADR 0016](docs/adr/0016-harnesses-are-build-steps.md)).
- **Heard equals value plus modulation**, across internal modulators, host modulation, per-note modulation and CC learn ([ADR 0017](docs/adr/0017-the-modulation-surface.md)).
- **Continuous pitch moves a reed along a family from cantilever scaling, never a bare frequency sweep** ([ADR 0018](docs/adr/0018-tuning-tables-and-a-reed-family.md)).
- **Savera models a pressure-blown Indian hand harmonium and nothing else.** No sample playback, no waveguide ([ADR 0019](docs/adr/0019-a-pressure-blown-indian-harmonium-and-nothing-else.md)).
- **The GUI is deferred; the parameter set is the interface** ([ADR 0020](docs/adr/0020-defer-the-gui.md)).
- **One signed, notarized, stapled `.pkg`, built locally and never in CI** ([ADR 0021](docs/adr/0021-distribute-as-a-notarized-pkg.md)).
- **Every build stamps its branch, commit and dirty state; plugin identity never varies with them** ([ADR 0022](docs/adr/0022-stamp-provenance-without-namespacing-identity.md)).
- **One note input port (CLAP, MIDI, MIDI-MPE; CLAP preferred) and one stereo output port** ([ADR 0023](docs/adr/0023-one-note-port-and-one-stereo-output.md)).

## Identifiers, which are permanent

A host writes these into project files, so changing one after release makes the plugin read as missing in every project that used it. They are doc-commented as permanent at their declaration sites.

| Thing                | Value                                               | Note                                                      |
| -------------------- | --------------------------------------------------- | --------------------------------------------------------- |
| Repository           | `savera`                                            | Matches the product name                                  |
| Product name (ASCII) | `Savera`                                            | Names files, binaries and bundles                         |
| Display name         | `Savera`                                            | No diacritic, so the two agree                            |
| CLAP id              | `com.catamountaudio.savera`                         | Catamount Audio vendor identity                           |
| Bundle identifier    | `com.cboone.savera`                                 | Signing identity, deliberately different                  |
| AU type              | `aumu`                                              | Music device: what Logic's software instrument slot loads |
| AU subtype           | `Svra`                                              | Distinct from fosforo's `Fsfr` and springer's `Sprg`      |
| AU manufacturer      | `Ctmn` / `Catamount`                                | Shared with fosforo and springer                          |
| CLAP `features[0]`   | `instrument`                                        | `features` is `{ instrument, synthesizer }`               |
| State magic          | `SVRA`                                              | The AU subtype, uppercased                                |
| Note input port      | id `0`, CLAP preferred, MIDI and MIDI-MPE supported | Dialects are rescan-gated                                 |
| Audio output port    | id `0`, two channels                                | Channel count is rescan-gated                             |
| Preset provider id   | `com.catamountaudio.savera.presets`                 | Hosts persist preset locations                            |

## Structure

Present today:

```text
.github/                   workflows, Dependabot, review instructions, community templates
docs/
  adr/                     settled decisions, 0001 to 0023
  design/                  the brainstorm, preserved verbatim
  notes/                   living documents: how things behave, what was measured
  plans/                   the permanent build plan; todo/ and done/ for phase plans
verification/              gitignored working material; never committed
package.json               pins the text lint tools only; zig build never reads it
```

What is planned is in the build plan's [Source layout](docs/plans/2026-09-13-savera-build-plan.md#source-layout), deliberately not copied here, where it would drift.

## Development

Everything that runs before Phase 1 is static analysis, and all of it also runs in CI.

```bash
npm ci                             # once per checkout; the pinned text tools
npm run format                     # Prettier writes
npm run format:check               # Prettier checks, exactly as CI runs it
npm run lint:md                    # markdownlint, with the relative-links rule
typos                              # spell check; words only when flagged
shellcheck --version && actionlint # shellcheck must be present or run: blocks go unchecked
gitleaks detect --no-banner        # secrets
```

`package.json` exists for the text tools and nothing else. Savera is Zig.

## Rules

Every one of these is here because omitting it causes **silent damage or a wrong pass**, not a legible error. The reasoning behind each is in the note or document named beside it.

- **The build plan never leaves `docs/plans/`.** The `commit` and `pr` skills move a plan from the `docs/plans/` root to `done/` once they judge its work complete, and neither implements the exemption for a permanent plan. After either runs, confirm `docs/plans/2026-09-13-savera-build-plan.md` is still where it was. See [skill deviations](docs/notes/skill-deviations.md).
- **`docs/design/` is preserved verbatim.** Prettier and markdownlint both exclude it, and nothing in CI would catch a hand edit. Never format, fix or reword the brainstorm, even where the build plan corrects it; a typo there is suppressed in `typos.toml`. See [linters](docs/notes/linters.md).
- **Run `npm ci` before any text check, run Prettier before markdownlint, and never pass `--fix` to `markdownlint-cli2`.** Without `npm ci` the Homebrew copies answer, at different versions. `--fix` rewrites every file its globs match. `lint-and-fix` is not used here for that reason. See [linters](docs/notes/linters.md).
- **No workflow that judges text or secrets gets `paths-ignore`.** A check must not be able to skip the change that governs it. See [CI workflows](docs/notes/ci-workflows.md).
- **A check that passes by finding nothing is not coverage.** The shell job does exactly that before Phase 1, a local `actionlint` without shellcheck on `PATH` does it always, and the TruffleHog job cannot fail at all, because the reusable workflow omits `--fail` ([cboone/gh-actions#111](https://github.com/cboone/gh-actions/issues/111)). gitleaks is the secret check that can go red. Say so rather than counting any of them. See [CI workflows](docs/notes/ci-workflows.md).
- **Nothing under `verification/` is ever committed.** Purchased sample libraries forbid redistribution, and a capture in history means rewriting commits. Committed reference audio is CC0 and lives in `fixtures/`.
- **Never commit a secret-shaped string, even a planted fake.** This repository is public. Plant against gitleaks in an untracked file only.
- **`docs/plans/done/` is a historical record; `docs/notes/` is living.** Never update a figure, citation or line number in a done plan. A stale figure in a note is repaired by a fresh measurement in place.
- **When a diff corrects a measured figure, grep the old value across the repository.** Figures are quoted across `AGENTS.md`, the ADRs, the build plan, the notes and config comments.

## Where the depth lives

The depth behind the rules is in [`docs/notes/`](docs/notes/README.md), one document per thing you might be about to do.

| Read before                                                                | Note                                               | Answers                                                                                                            |
| -------------------------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| editing `.github/workflows/`, `.github/dependabot.yml`, or reading checks  | [CI workflows](docs/notes/ci-workflows.md)         | the jobs and their rollup rows, why no `paths-ignore`, the pinned gh-actions release, timeouts, moving a pin       |
| running or configuring Prettier, markdownlint, typos, actionlint, gitleaks | [linters](docs/notes/linters.md)                   | the pins and why not Homebrew, what is excluded and the controls proving it, the relative-links rule, typos tables |
| doing by hand what a catalog skill should do                               | [skill deviations](docs/notes/skill-deviations.md) | what was re-derived against which catalog issue, and which installed skills were scoped or replaced                |

## Maintaining this file

**This file is loaded into every session in full, and `docs/notes/` is not.** That is the basis for deciding where something goes. Keep it under 30,000 characters, fosforo's refusal threshold, which sits 25% under the 40,000-character size at which Claude Code starts warning about a memory file. Nothing automates that check here; each phase pull request measures it with `wc -c AGENTS.md` and confirms `CLAUDE.md` is still a symlink to this file. Before adding something, ask:

- **Does an agent that has not opened anything need it?** A rule whose omission causes silent damage or a wrong pass belongs in [Rules](#rules). One whose omission causes a compile error or a failing test does not; the compiler is already saying so.
- **Is it a settled decision?** Then it is an ADR, and [Non-negotiables](#non-negotiables) gets one line pointing at it.
- **Is it a measurement, a refusal, or how some area behaves?** Then it is a note in [`docs/notes/`](docs/notes/README.md), and nothing is added here. This is the common case.
- **Is it what has landed?** Then it belongs in the build plan's phase outcomes, or in `CHANGELOG.md`. [Current state](#current-state) is orientation, not a record.
