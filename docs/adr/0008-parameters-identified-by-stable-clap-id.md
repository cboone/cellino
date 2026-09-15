# 0008. Parameters are identified by stable `clap_id`

**Status:** Accepted

## Context

A host writes parameter identity into every saved project and every automation lane. If identity is a position in a list, reordering or inserting a parameter silently repoints automation in every project that used it. CLAP avoids this: `clap_param_info` carries an explicit `clap_id`, and hosts persist that id rather than an ordinal.

Identity is not the only thing that freezes at release. Parameter ids, the modulatable and per-note flags, ranges, port channel counts and the note port's dialects can only change under a full rescan while the plugin is deactivated (`clap/ext/params.h` and `clap/ext/note-ports.h`, per the [Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)). A plugin whose whole premise is that every physical quantity is modulatable cannot discover its flag policy after projects exist.

## Decision

Assign every parameter a `clap_id` once, in an enum, doc-commented as permanent. Ids are never reused, never renumbered, and never derived from position. Groups are numbered with gaps. Every flag and range that needs a deactivated rescan to change is declared before the first release.

## Consequences

**Groups have room to grow.** Global 1 to 99, Air 100 to 199, Pump 200 to 299, Reed 300 to 399, Banks 400 to 499, and so on to Voice at 1300 ([Parameters and the modulation surface](../plans/2026-09-13-savera-build-plan.md#parameters-and-the-modulation-surface)). Later groups, such as the coupler, modulators and macros, have their ranges reserved with no ids assigned, and the Registration id is reserved before Phase 3 assigns the rest.

**Ranges are declared at their widest.** Every continuous physical parameter is declared at the widest range the engine will ever accept, the beyond-acoustic range included, with the realistic value as its default. The stepped Range parameter clamps the engine to the acoustic range at runtime, so unlocking the extended range in Phase 9 changes no declared bound.

**Flags are set generously and cleared never.** Setting a modulation flag the engine does not yet honour is harmless, since the engine ignores `PARAM_MOD` for that id until it can act on it. Clearing one later is the expensive operation. The per-parameter flag policy lives in [ADR 0017](./0017-the-modulation-surface.md); the per-parameter values live in the Phase 3 plan.

**AUv2 ordering is a separate problem and does not go away.** clap-wrapper otherwise sorts parameters by id, and Logic presents them in the order the Audio Unit reports. `clap.plugin-auv2-param-ordering` is implemented from the first release, and `module` paths give Logic's generic view its clumps and Bitwig its tree.

**Stable ids are a claim to check, not an intention.** Phase 3 carries a unit test that the flags read back from `clap.params` for every id match the declared policy, a source canary over the id enum, and the by-hand check that an automation lane still binds to the right parameter after a rebuild.
