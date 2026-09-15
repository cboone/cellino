# 0001. macOS on Apple Silicon only

**Status:** Accepted

## Context

This is a single-developer project, and every axis of platform variation multiplies the decision lattice rather than adding to it: several audio-thread contracts, several plugin-format packaging stories, several signing regimes, several installer formats.

The target hosts settle most of it before preference gets a say. Savera's first acceptance host is Logic Pro, which loads it as a software instrument through an Audio Unit and runs on macOS and nowhere else. REAPER and Bitwig load it as a CLAP. A Windows or Linux build would serve two of those three hosts on platforms where the first cannot exist, at the cost of a second audio-thread contract and a second distribution story.

The instrument also leans on macOS mechanisms directly. [The solver](../plans/2026-09-13-savera-build-plan.md#the-solver) sets flush-to-zero through bit 24 of the ARM64 floating-point control register around `process()`, and the CPU gate is measured on an Apple performance core. Both are Apple Silicon facts, not portable abstractions.

## Decision

Target macOS on Apple Silicon as the primary and only platform for the plugin. The deployment target is macOS 11.0. Treat Intel Mac support as a retained option, not a commitment.

## Consequences

Committing to one platform removes whole categories of work rather than merely reducing them:

- **One audio-thread contract.** Core Audio's rules, rather than the intersection of several platforms' differing and differently strict rules.
- **One signing and distribution story.** One code-signing regime, one notarization flow, one installer format ([ADR 0021](./0021-distribute-as-a-notarized-pkg.md)), one binary architecture.
- **One packaging target beyond CLAP.** AUv2, because that is what Logic loads ([ADR 0003](./0003-author-clap-project-outward.md)).

**The deployment target is stated in four places that must agree:** `build.zig`, `cmake/CMakeLists.txt`, `macos/Info.plist` and `packaging/distribution.xml`. None of them exists before Phase 1, and each is written to the same value.

**Some of the build runs on Linux, and that is not a portability promise.** Savera has no GUI and so no zig-objc, links no Apple framework, and its CLAP dependency has no build function, so `zig build render`, `zig build test` and `zig build smoke` build on Linux. The non-macOS early return in `build.zig` guards only signing, the `install-*` steps and the CMake step ([Build architecture](../plans/2026-09-13-savera-build-plan.md#build-architecture)). That path exists so the oracle can run beside the Ubuntu `python` job in CI. It does not make the plugin a Linux product, and nothing ships from it.

The cost is that users of CLAP hosts on Windows and Linux are not served. [ADR 0005](./0005-a-pure-model-core-behind-a-seam.md) keeps the physical model in pure Zig that names no CLAP type and no platform API, so if this decision is ever revisited the work is porting the plugin shell and the FPCR handling, not the instrument.
