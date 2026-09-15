# 0023. One note input port and one stereo output port

**Status:** Accepted

## Context

An instrument's ports are part of its identity to a host. A note port's supported dialects and preferred dialect, and an audio port's channel count, can only change under a full rescan while the plugin is deactivated (`clap/ext/note-ports.h` and `clap/ext/audio-ports.h`, per the [Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)), so they have to be right before any project saves them.

The inputs Savera needs arrive in different dialects in different hosts. CLAP note events carry note ids and note expressions, which per-note modulation and key depth depend on. Controllers that drive the bellows arrive as raw MIDI everywhere. Through clap-wrapper in Logic, channel pressure arrives as a channel-wide pressure expression with the key wildcarded, polyphonic key pressure arrives per key, and pitch bend arrives as a tuning expression at a fixed two semitones with no RPN handling.

## Decision

One note input port, id `0`, declaring the CLAP, MIDI and MIDI-MPE dialects with CLAP preferred. One stereo audio output port, id `0`. Dialects, preference and channel count are fixed before the first release. clap-wrapper's fixed two-semitone bend translation in Logic is accepted, and glides there go through the two-key mapping and the tuning tables ([Note input](../plans/2026-09-13-savera-build-plan.md#note-input)).

## Consequences

**CLAP preferred gains note ids where the host has them.** Voices are matched on the full `(port, channel, key, note_id)` tuple with wildcards, following CLAP's own rule in `clap/events.h`: a `-1` in any part of the tuple matches a voice with any value there. So a note-off that carries no note id releases every voice sounding on that port, channel and key. When notes do not overlap on a key, that is the one voice a host issuing ids only at note-on expects to release. When two notes overlap on the same key, only host note ids can release them independently; without ids, both release together, which is the specified behaviour, not a lost voice. The same rule applies to raw MIDI note-offs, which carry no id. Phase 4's host harness plants the duplicate-key case, with and without note ids.

**Note expressions have defined destinations.** `PRESSURE` and `EXPRESSION` feed key depth and per-voice pressure ([ADR 0013](./0013-keys-are-continuous-valves.md)); `TUNING` is the per-note pitch input in semitones ([ADR 0018](./0018-tuning-tables-and-a-reed-family.md)); `BRIGHTNESS` maps to the aperture scale. `VOLUME`, `PAN` and `VIBRATO` are mapped or ignored explicitly, and the choice is doc-commented where it is made.

**MIDI is parsed, not tolerated.** `src/clap/midi.zig` turns controller changes, channel pressure and RPN sequences into engine inputs from Phase 1, because the bellows is played from raw controllers in every host ([ADR 0017](./0017-the-modulation-surface.md)).

**Endings are explicit.** `NOTE_CHOKE` releases every reed of the voice immediately, and `reset` and `deactivate` release everything. Because a voice outlives its note-off by its ring-down, the engine emits `CLAP_EVENT_NOTE_END` when a voice's last reed falls below the audible-decay threshold, which polyphonic modulation in a host requires.

**Logic's bend limit is accepted rather than fought.** Glides in Logic use the two-key mapping, which works because polyphonic key pressure is delivered per key, and the tuning tables. Continuous pitch through bend is a CLAP-host path.

**The output is stereo from the first release even though it carries one signal.** Both channels carry the same monopole ([ADR 0014](./0014-flow-derivative-through-an-enclosure-filter.md)). Declaring two channels now costs nothing, and declaring one would make the jali morph and any later dipole model a breaking change for every saved project.
