# 0013. Keys are continuous valves

**Status:** Accepted

## Context

On a harmonium, a key opens a pallet over a reed. Loudness lives in the bellows pressure, not in how hard the key is struck, and nearly every sampled harmonium gets this wrong by mapping velocity to level.

What a player controls with the key is depth. Puranik and colleagues (ICASSPW 2025) report that key depth, not velocity, sets loudness and onset shape, and pressing a key partway is how players fake the pitch ornaments a fixed reed cannot make. Partial pallet opening measurably lowers pitch: Coyle, Behrens and Cottingham (2009), as presented in Cottingham's CCRMA 2013 notes, measured up to 2.5 Hz on a 96 Hz reed at a quarter depression, about 45 cents at that pitch, on what the slides' context indicates is an accordion reed. No harmonium figure exists ([Findings](../plans/2026-09-13-savera-build-plan.md#findings-that-changed-the-plan)).

## Decision

Keys are continuous valves. Key depth sets the pallet orifice area through a trajectory whose rate comes from velocity. Velocity drives only the pallet's opening rate and the key noise ([Keys, coupler, drones, scale changer, tremolo](../plans/2026-09-13-savera-build-plan.md#keys-coupler-drones-scale-changer-tremolo)).

## Consequences

**Depth needs a source, and the host decides which exists.** It comes from a per-note pressure expression, from polyphonic aftertouch, or, when the host has neither, from the note-on and note-off events alone: full depth while the key is held and none after it is released, selected by the stepped Key Depth Source parameter. **That fallback never reads velocity.** A note-on's velocity sets how fast the pallet reaches full depth and how loud the key noise is, never how deep it goes, so pitch and loudness stay velocity-independent in every depth source. Through clap-wrapper in Logic, polyphonic key pressure arrives per key, which is what makes depth playable there ([ADR 0023](./0023-one-note-port-and-one-stereo-output.md)).

**Partial depth does three things together**: the note comes in quieter, fades in rather than starting, and sits slightly flat. The null arm of the by-hand check is full depth, where none of the three happens.

**The pitch pull is modelled as an acoustic load, because the obvious model gets the sign wrong.** A partly closed pallet reduces the pressure difference across the reed, and by the single-mode reed's own pressure-to-pitch law ([ADR 0009](./0009-single-mode-reed-behind-a-reed-seam.md)) a smaller pressure difference would raise the pitch, not lower it. So the pull comes from the partly closed pallet changing the near-field inertance and end impedance the reed sees, which is where Cottingham's group attributes the chamber-geometry bends measured beside it. Phase 5 gates the sign of this pull separately from the pressure-to-pitch drift, so the two cannot mask each other.

**The only figure is from an accordion reed, so the pull is judged qualitatively.** The corpus releases and the by-hand protocol judge its size; the sign is the gate.

**Releasing a key closes two things at once.** Because the pallet is downstream, closing it stops the flow and closes the reed's path into the cabinet together, so the audible release is the product of the reed's mechanical ring-down and the closing pallet's transmission. Phase 5 measures which dominates against the per-key release samples in the corpus. The steal reserve, the `NOTE_END` threshold and the perceptual ring-down target are sized from that audible decay, while `τ = Q / (π f₀)` remains an analytic invariant on the reed state alone.

**Nothing else reads velocity.** The first by-hand verification step confirms that neither pitch nor onset depends on it.

**Depth is also a pitch input.** The two-key mapping interpolates the sounding pitch between two held keys from their depths ([ADR 0018](./0018-tuning-tables-and-a-reed-family.md)), which only works because depth is continuous.
