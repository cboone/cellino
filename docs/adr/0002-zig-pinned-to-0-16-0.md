# 0002. Zig, pinned to 0.16.0

**Status:** Accepted

## Context

The plugin format is a plain C API ([ADR 0003](./0003-author-clap-project-outward.md)), and the instrument is a per-sample nonlinear solver whose inner loop must not touch the heap, take a lock, or be guessed about. The language that fits is one with frictionless C interoperability, no runtime in the way of the audio thread, and control over floating-point behaviour.

Zig provides what this problem needs:

- **C interoperability with no binding layer.** The entire host interface is a header, and a header becomes a Zig module with no glue to maintain ([ADR 0004](./0004-clap-bindings-via-translate-c.md)).
- **An explicit-allocator convention.** If nothing on the audio path is handed an allocator, "does the audio path touch the heap" is a fact about the call graph rather than a hope ([ADR 0007](./0007-no-allocation-on-the-audio-thread.md)).
- **`comptime`.** The reed tables over geometry, the capacity derivation and the ABI layout assertions are computed or checked at compile time.
- **Scoped float modes and `@Vector`.** The solver runs in `.strict` float mode so its residual is identical in every optimize mode, and lanes across independent reeds can use SIMD arithmetic.

The cost is that Zig is pre-1.0 and moves. On 2026-09-13, 0.16.0 was the current stable release at `/opt/homebrew/bin/zig`, and 0.17 existed only as a development snapshot ([Environment status](../plans/2026-09-13-savera-build-plan.md#environment-status)).

## Decision

Use Zig, pinned to 0.16.0. The pin lives in `build.zig.zon` as `minimum_zig_version`, and CI reads that file as the single source of truth rather than restating the version in a workflow.

Treat compiler upgrades as scheduled, deliberate work, never as churn absorbed mid-feature.

## Consequences

One string is read everywhere the toolchain is chosen. It cannot drift, because there is only one of it, and a Zig CI workflow cannot exist before the file it reads, which is why `ci.yml` arrives with `build.zig.zon` in Phase 1.

0.16.0 shapes the build concretely, as springer and fosforo measured:

- `@cImport` is deprecated in favour of `b.addTranslateC()`, which forces [ADR 0004](./0004-clap-bindings-via-translate-c.md).
- File I/O moved under `std.Io`, which is why `src/platform/io.zig` holds the one `std.Io` instance, adapted from fosforo.
- Runtime indexing into a vector is forbidden, so each reed lane's table lookups are scalar loads into an array that is then coerced.
- **The archive writer pads member names to four bytes rather than eight**, so whether an object inside a `.a` lands on the eight-byte boundary Apple's linker requires depends on the size of the symbol table. springer's archive was refused with `not 8-byte aligned`; fosforo's happened to align. Savera hands CMake an object file built with `b.addObject`, which removes the archive format from the question. This workaround is tied to the pin and is revisited when the pin moves.

**0.17 will move things again**, and the build plan records the known hazards as an accepted risk ([Risks](../plans/2026-09-13-savera-build-plan.md#risks)): the float-mode builtin is slated for removal in favour of typed reals, and translate-c changed backends. Scoping the float mode per file and coercing vectors to arrays are the migration-safe idioms, and new code uses them.

Essentially every third-party Zig audio project predates this release and does not build against it, so the dependency surface stays minimal and each dependency is verified against 0.16.0 before it is adopted.
