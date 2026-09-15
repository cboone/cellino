# 0018. Tuning tables plus per-note tuning; continuous pitch moves along a reed family

**Status:** Accepted

## Context

A free reed's pitch is set by its length and thickness when it is built, and fixed thereafter. That is the central limitation of the harmonium in Hindustani practice: it cannot glide, and it plays twelve-tone equal temperament in a tradition built on the 22 shruti. A model can do both, and continuous pitch and microtonality are first-class goals of this project rather than stretch features.

The obvious implementation is wrong. Sweeping a reed's eigenfrequency while holding its mass, stiffness, pumped section and aperture geometry fixed produces a reed no instrument contains, and Puranik's lumped adaptation already needed per-note retuning of the non-playing parameters to stay stable ([Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)). A bare frequency sweep would walk the solver out of the region where it is known to behave.

There is also no host standard to lean on. Draft `clap.tuning` has no shipping host, and MTS-ESP, which does work in Logic, is a C++ client library.

## Decision

Tuning is internal tables plus per-note tuning expressions. Continuous pitch moves a reed along a one-parameter family derived from cantilever scaling, never a bare frequency sweep, and the two-key mapping drives that same family. MTS-ESP is deferred ([Tuning and continuous pitch](../plans/2026-09-13-savera-build-plan.md#tuning-and-continuous-pitch)).

## Consequences

**The first release is twelve-tone equal temperament at A 440**, with the reference pitch as a parameter and the tuning parameter ids reserved.

**The family keeps every parameter consistent.** A reed's eigenfrequency, mass, stiffness, pumped section and aperture geometry are tied together by `f ∝ t / L²`, and `src/model/family.zig` defines a one-parameter family along that law, so a glide moves every non-playing parameter consistently and stays inside the region Puranik found stable. The same family generates a bank's reed table from the one measured reed ([ADR 0009](./0009-single-mode-reed-behind-a-reed-seam.md)). A swept-frequency oracle vector is the Phase 9 gate, and a stability sweep over the family is already a Phase 2 oracle vector.

**The keyboard-native path is the continuous harmonium mapping.** West, Puranik, Scavone and Wanderley built it into a sensor-retrofitted harmonium: the sounding pitch is interpolated between two held keys, `p = (1 − w′) p₁ + w′ p₂` with `w′ = min(1, w₂ / w₁)` from the two key depths, and the amplitude follows the key-press weight. `p` is a pitch, so this path drives the same reed family as the glide. It works in Logic because polyphonic key pressure is delivered per key ([ADR 0013](./0013-keys-are-continuous-valves.md)), which matters because clap-wrapper translates pitch bend there to a fixed two-semitone tuning expression ([ADR 0023](./0023-one-note-port-and-one-stereo-output.md)).

**The mapping's zero-depth cases are open, and Phase 9 closes them before implementing it.** As published, `w′` divides by the first key's depth `w₁`, and ADR 0013 allows a held key at zero depth, so two held keys at zero depth give `0 / 0` and a zero-depth first key under a pressed second key has no defined pitch. Phase 9 defines the pitch and weight in both cases, keeps `w′` finite and continuous as `w₁` approaches zero, and covers both in the swept-frequency oracle vector with an explicit NaN rejection, so no non-finite value can reach the reed family.

**The beyond-acoustic phase adds the tables.** Temperament tables, Gandhar tuning after Chimote and Oke's 22 shruti with per-key lower and upper selection, twelve per-key offsets, and per-note tuning expressions in semitones, all in Phase 9.

**Draft `clap.tuning` is not implemented**, because no host ships it, and reaching a draft extension is a deliberate act ([ADR 0004](./0004-clap-bindings-via-translate-c.md)).

**MTS-ESP is deferred, not refused.** It would work in Logic, but its client adds a C++ source to a Zig build that otherwise has none outside clap-wrapper. It is recorded as a deferred issue, and internal tables and per-note tuning expressions come first.
