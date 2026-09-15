# 0022. Stamp build provenance, and leave plugin identity alone

**Status:** Accepted

## Context

This repository is normally checked out as several worktrees on several branches, and all of them compete for one pair of directories under `~/Library/Audio/Plug-Ins/`. Building and installing are separate acts and only the second decides what a host loads, so the installed bundle belongs to whichever worktree copied last.

The resulting failure is silent and reads as a pass. fosforo recorded it repeatedly: a feature "verified" against an installed build from another branch, and a two-week-old Audio Unit that Logic loaded, rendered and passed. Its [ADR 0018](https://github.com/cboone/fosforo/blob/1317e2b752f7f7d44db9bb7745200ca6e59dde83/docs/adr/0018-stamp-provenance-without-namespacing-identity.md) separates the problem into a path collision, which is what actually went wrong, and an identity collision, which never has, and decides to make the first visible rather than to namespace the second.

Savera makes the hazard worse, not better. It is the third plugin installing into the same `~/Library/Audio/Plug-Ins` tree as fosforo and springer, so the multi-worktree hazard is live from the first install, and nearly every verification row in its build plan is judged in a running host.

## Decision

Every build stamps the branch, commit and dirty state it came from, and nothing about the plugin's identity varies with them. This ADR adopts fosforo's decision.

## Consequences

**Provenance is stamped at configure time and read back by one script.** `build.zig` runs `git rev-parse` through `b.runAllowFail` and passes the result through build options; `src/build_info.zig`, adapted from fosforo, composes the strings; `scripts/read-provenance` is the one implementation of reading them back out of a bundle.

**It takes two `git rev-parse` calls, never one.** `git rev-parse --abbrev-ref HEAD --short HEAD` prints the branch name twice, because `--abbrev-ref` is sticky across every ref that follows it, and a provenance line that is wrong while looking well-formed is worse than a missing one.

**The marker's survival is asserted, not reasoned about.** It stays in the binary by being logged from `plugin.init`, so the linker does not strip it as unreferenced. That is a claim about the optimizer, so CI runs `scripts/read-provenance --check` against built bundles rather than trusting it. fosforo found Debug and ReleaseFast emit the marker differently, and its reader takes the longest match for that reason.

**Every install says what landed.** Every `install-*` step builds exactly what it installs, copies it through the one `scripts/install-plugins` implementation, and prints the hash and provenance of the installed bundle. Every other step stays in the worktree. Each by-hand verification step begins with a provenance check of the build under test.

**Identity is not namespaced.** No per-worktree CLAP id, no hashed AU subtype, no suffixed display name. Those reach the identifiers the build plan marks permanent ([Identifiers, which are permanent](../plans/2026-09-13-savera-build-plan.md#identifiers-which-are-permanent)), which is the one category of change that cannot be undone after release. The version string carries provenance instead, as free metadata.

**The Audio Unit is not installed as a symlink.** fosforo implemented that and measured that macOS does not register a symlinked component at all, with no error anywhere. The install path stays exclusive for the Audio Unit, which is why `~/Library/Audio/Plug-Ins` is an exclusive resource in the build plan. A CLAP can be loaded out of a worktree through `CLAP_PATH` in hosts that honour it, which relaxes the constraint for that format only.

**Two worktrees still cannot load Audio Units at once.** The stamp makes the collision visible rather than preventing it, which is the failure mode that has actually cost time. If simultaneous Audio Units are ever needed, a development channel is the design, and it needs a new ADR because it reaches identifiers this one leaves alone.
