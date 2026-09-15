# 0020. The GUI is deferred; the parameter set is the interface

**Status:** Accepted

## Context

Savera's first release carries about fifty parameters and the full instrument well over a hundred. Presented in Logic's generic parameter list that is a lot to scroll, and it is the strongest argument for pulling an interface forward ([Risks](../plans/2026-09-13-savera-build-plan.md#risks)).

It is still the wrong thing to build early. An interface cannot be designed until the parameter set is settled, and the parameter set is not settled until the engine that consumes it works: the air path, the calibration pass and the full instrument each add or reshape parameters. Building the interface first means building it more than once.

Every parameter is reachable and automatable through the host's generic list in the meantime. The instrument is fully playable without an interface, from a keyboard, an expression pedal and learned controllers; it is merely unpleasant to configure.

## Decision

No GUI until the instrument is done. The parameter set is the interface until then. When the GUI arrives it is native AppKit controls or a Metal-rendered panel, decided at that time and constrained by [ADR 0006](./0006-reject-webview-ui.md).

## Consequences

**The generic list is made as usable as it can be.** `module` paths give Logic's generic view its clumps and Bitwig its tree, and `clap.plugin-auv2-param-ordering` fixes the order Logic presents rather than leaving it sorted by id ([ADR 0008](./0008-parameters-identified-by-stable-clap-id.md)). Those are the mitigation, and the parameter-count risk is accepted with them.

**Deferring costs nothing structural.** Parameters are keyed by stable `clap_id`, so nothing an interface needs is foreclosed by building it later, and no parameter has to exist only for an interface.

**What the GUI shows is decided when it lands.** Whether it shows state-variable plots, which would reserve fosforo's ring buffer and display-link machinery, is part of that decision; nothing is reserved now. The GUI is listed with the optional Phase 10 work, each item gated by a measurement, and the parameter-count risk is the measurement most likely to pull it forward.

**The cost is real.** Until then, configuring Savera means scrolling a long generic list, and presets carry most of the burden of a good starting sound: the Registration parameter and `.aupreset` files in Logic, preset discovery in CLAP hosts ([ADR 0017](./0017-the-modulation-surface.md)).
