# 0003. Author once as a CLAP instrument; clap-wrapper projects it to an AUv2 `aumu`

**Status:** Accepted

## Context

Logic Pro's software instrument slot loads Audio Units of type `aumu`, and Logic is the first host this instrument has to play in. Authoring an Audio Unit directly means Apple's AU framework, Objective-C++ and a component packaging story that exists nowhere else.

CLAP is a plain C API with no inheritance, no framework and no code generation, and it is the format REAPER and Bitwig load directly. clap-wrapper projects a CLAP outward into other formats, including AUv2, and its clap-first entry point wraps a plugin that was authored as a CLAP rather than the reverse. fosforo and springer both ship this way.

Neither sibling exercised an instrument. fosforo is an `aufx` effect and springer an `aumi` MIDI processor, so an `aumu` with note input through clap-wrapper is unmeasured in this family. That is the largest integration unknown in the project.

## Decision

Author the plugin exactly once, as a CLAP instrument with `features` of `{ instrument, synthesizer }`. Use clap-wrapper to project it outward to an AUv2 of type `aumu`, subtype `Svra`, manufacturer `Ctmn`, and pass `AUV2_INSTRUMENT_TYPE "aumu"` to `make_clapfirst_plugins` explicitly. Read the type back out of the built plist as the control. `zig build` alone produces a loadable `Savera.clap`; CMake exists only to run clap-wrapper and emit `Savera.component`.

## Consequences

**The explicit type makes clap-wrapper's `features[0]` mapping unreachable, so no grep for its warning is a control.** In clap-wrapper v0.16.0, `build-helper.cpp` checks an explicit `AUV2_INSTRUMENT_TYPE` before it looks at `features[0]`, and `aumu` is also what it falls back to, with a warning, for an unrecognised feature ([Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)). The helper's "can't determine instrument type" warning therefore cannot fire in the shipping configuration. Two positive controls guard the type instead, chosen so neither is disarmed by the other: a unit test that `features[0]` equals `CLAP_PLUGIN_FEATURE_INSTRUMENT` as restated in `src/clap/c.zig`, and a CI step that runs `plutil -extract AudioComponents.0.type raw` on the built component's `Info.plist` and requires `aumu`. Each is proven in Phase 1 by planting a wrong value and confirming it goes red.

**clap-wrapper is pinned by commit, to `1cca996e96f29ab2be7ae9f8cfe532bbc92e1dd6` (2026-08-08, the v0.16.0 release) or later**, for the three pull requests springer records: full AUv2 MIDI 1.0 and 2.0 input with dialect negotiation, the silent facade, and cleared silence buffers. The reason is written in `cmake/CMakeLists.txt` beside the pin. That file also sets `CMAKE_OSX_DEPLOYMENT_TARGET`, `CMAKE_CXX_STANDARD 17` and `CLAP_WRAPPER_DOWNLOAD_DEPENDENCIES TRUE`, which clap-wrapper does not set for a consumer.

**The wrapper's generated plist is rewritten before signing** by scripts with a `--check` mode, adapted from fosforo's, and CI asserts the result independently, because a rewriting step cannot detect its own absence and signing seals the plist.

**The wrapper carries recorded deficits, and they are its, not the plugin's.** At v0.16.0 it hard-codes `SupportsTail()` to false while implementing `GetTailTime()`, so the Audio Unit reports no tail ([ADR 0011](./0011-fixed-internal-sample-rate.md)). It implements no preset extension, so presets reach Logic through a stepped Registration parameter and shipped `.aupreset` files. It delivers no parameter modulation and translates pitch bend to a fixed two-semitone tuning expression ([ADR 0017](./0017-the-modulation-surface.md), [ADR 0023](./0023-one-note-port-and-one-stereo-output.md)).

**`clap-validator` runs on both bundles**, the Zig-built CLAP and the wrapper-built one, so the seam between them is checked rather than assumed.

**`auval` is expected to be no help.** fosforo's bundle declares `aufx`, Logic loads it, and `auval` still does not list it, so the invisibility is a clap-wrapper property and Savera inherits it. Logic is the only complete check of the Audio Unit, which is why Phase 1 proves the `aumu` path against a placeholder sine voice before any reed code depends on it.
