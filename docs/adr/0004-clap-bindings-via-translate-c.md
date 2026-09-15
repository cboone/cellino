# 0004. CLAP bindings from translate-c over a preprocessing step

**Status:** Accepted

## Context

The CLAP headers are the entire host interface, and they must become Zig types that match the C ABI exactly. A wrong field offset is not a compile error; it is a plugin that reads the wrong bytes out of a host-supplied struct on the audio thread.

`@cImport` is deprecated in Zig 0.16.0 ([ADR 0002](./0002-zig-pinned-to-0-16-0.md)), which forces the question rather than leaving it to taste. Hand-written bindings are worse: the headers are large, they change between CLAP versions, and every transcribed field is a chance for a silent layout mismatch.

Feeding translate-c the CLAP headers directly fails too. CLAP's headers include each other through relative paths, and translate-c mishandles `#pragma once` when the same header is reached under two spellings of its path, producing redefinition errors that plain clang does not.

CLAP 1.2.10, the newest tag, is what fosforo vendors through `build.zig.zon`, and Savera pins the same.

## Decision

Generate the bindings with `b.addTranslateC()` in `build.zig`, fed a preprocessed umbrella header. `src/clap/clap_all.h` includes exactly what the plugin uses; a `zig cc -E -P` step preprocesses it; translate-c consumes the result.

Assert the result. `src/clap/c.zig` carries `comptime` `@sizeOf` and `@offsetOf` checks over every CLAP struct that crosses the ABI, and they run inside `zig build test`.

## Consequences

**Object-like macros do not survive preprocessing, so the ones that matter are restated.** `CLAP_PLUGIN_FEATURE_INSTRUMENT` is the one that matters most: [ADR 0003](./0003-author-clap-project-outward.md)'s unit test compares `features[0]` against the restated constant in `src/clap/c.zig`, so the restatement is itself load-bearing and is tested, not transcribed and forgotten.

**The umbrella header is the plugin's dependency surface, written as a list.** CLAP's draft extensions are not in `clap/clap.h`; they live only in `clap/all.h`, so reaching one is a deliberate act. Draft `clap.tuning` is one Savera deliberately does not reach for ([ADR 0018](./0018-tuning-tables-and-a-reed-family.md)).

**The wrapper's headers are checked against the Zig side's.** clap-wrapper downloads its own copy of the CLAP headers when CMake configures. A configure-time check asserts that the version it resolved matches the tag `build.zig.zon` pins, so the layout assertions guard the header set the Audio Unit is actually built against, not only the one Zig translated.

**The comptime assertions are the point, not a formality.** A binding that compiles is not a binding whose layout is right, and verifying them is part of the verification program: planting a deliberately wrong `@offsetOf` must turn the build red, which asserts a property of the check rather than of the code.
