# 0017. The modulation surface

**Status:** Accepted

## Context

Savera's ultimate goal is a highly modulatable synthesizer with a peti's sound. Every physical quantity the engine reads per sample is a parameter, and every such parameter is meant to be modulatable: the model is the synthesizer, and no separate synth engine is bolted on ([What we are building](../plans/2026-09-13-savera-build-plan.md#what-we-are-building)).

Hosts deliver modulation very unevenly, and the unevenness was measured rather than assumed ([Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)). Logic through clap-wrapper v0.16.0 delivers no parameter modulation at all: the only `param_mod` reference in its AUv2 process path is outbound and dropped, and the modulatable flags are never read. REAPER's modulators are destructive, writing parameter values rather than modulation. Only Bitwig delivers CLAP modulation and per-note expressions. Meanwhile the loudness control itself, bellows pressure, arrives from expression pedals, breath controllers, channel pressure and the modwheel as raw MIDI in every dialect and every host.

The flags that make a parameter modulatable are rescan-gated and freeze at release ([ADR 0008](./0008-parameters-identified-by-stable-clap-id.md)).

## Decision

The heard value of a parameter is its value plus its modulation. Internal modulators route to physical targets. Host and per-note modulation flags are set from the first release. MIDI CC learn is the fourth surface and ships with them, pushing each controller-driven change back to the host as a `PARAM_VALUE` event flagged `DONT_RECORD`. Bitwig verifies delivery.

## Consequences

**The flag policy is stated now, and the values land in Phase 3** ([Parameters and the modulation surface](../plans/2026-09-13-savera-build-plan.md#parameters-and-the-modulation-surface)). Every continuous physical parameter the engine could read per sample carries `IS_AUTOMATABLE | IS_MODULATABLE`. Those that are per-voice in the engine, the reed and key groups, also carry the per-note, per-key and per-channel modulation and automation flags. Structural parameters such as Range, Bellows Source, Pump Mode, Key Depth Source, Coupler, Cabinet Preset, Registration, Temperament and Steal Mode are stepped, several are enums, and none is modulatable.

**Modulation composes and is never stored.** A polyphonic modulation already includes the monophonic signal. The internal routing matrix's outputs enter the same `value + mod` sum as host modulation, so the two compose. No modulation amount is ever written into state, and the Phase 3 gate checks that a saved Bitwig project records none.

**Internal modulators are what works everywhere.** Two LFOs with free and tempo-synced rates, two per-note envelopes, an eight-slot routing matrix whose targets are the modulatable parameters, and four macros, all in Phase 9, with Bellows as the first target. The pump generator, which ships in the first release, is itself a modulator with a stroke shape. The by-hand verification runs the matrix in Logic precisely because Logic has no host modulation to confuse it with.

**CC learn is how the bellows is played.** `src/clap/midi.zig` parses raw MIDI from Phase 1. The learned map starts from CC 11 to Bellows with CC 2 as the alternate, and it is carried in state as parameter id, controller number and channel triples. Pushing changes back flagged `DONT_RECORD` keeps automation and live control from fighting. The Audio Unit path never sees that flag, so the by-hand check records any host that writes automation for a learned controller anyway as a host refusal.

**Delivery is verified where it can be delivered.** Bitwig is an acceptance host, acquired in Phase 3, where its gate needs it. A mechanism a host refuses is recorded with the host or wrapper behaviour that causes it, not worked around silently.

**Presets reach the two formats differently, and deliberately.** CLAP hosts get a preset-discovery factory under `com.catamountaudio.savera.presets` plus `clap.preset-load`. clap-wrapper v0.16.0 implements no preset extension, so Logic gets the stepped, non-modulatable Registration parameter, whose id is reserved now, plus shipped `.aupreset` files generated from saved state. The duplication is a recorded wrapper deficit.
