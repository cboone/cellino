# 0006. No WebView UI

**Status:** Accepted

## Context

Embedding a WebView is a common way to build plugin interfaces, and the appeal is real: web layout is well understood, the tooling is mature, and the same interface can be developed outside the host.

The costs land in the wrong places for this project. A WebView loads a browser engine into the host's process. It brings a JavaScript runtime, an asynchronous message boundary between the interface and the audio engine, a bundle of web assets to ship and sign, and a rendering path whose scheduling the plugin does not control.

Whatever Savera's interface eventually becomes, it is controls over a physical model's parameters, and perhaps plots of its state variables: reed displacement, chamber pressure, flow. That is a drawing problem, and the second half of it is a real-time drawing problem that fosforo already solves natively.

## Decision

No WebView. When a GUI arrives it is native AppKit controls or a Metal-rendered panel, decided at that time.

## Consequences

Nothing is bundled that has to be signed, sandboxed, notarized or kept current for security reasons beyond the plugin binary itself. Under [ADR 0001](./0001-macos-on-apple-silicon-only.md) there is exactly one windowing system to target, so the portability argument for a WebView does not apply.

There is no asynchronous boundary between the interface and the engine to design, and no serialization format between them to version.

This decision is about the mechanism, not the schedule. [ADR 0020](./0020-defer-the-gui.md) defers the GUI itself; this one constrains what it may be built from when it arrives. Whether that GUI shows state-variable plots, which would reserve fosforo's ring buffer and display-link machinery, is decided then; nothing is reserved now.

The cost is that the eventual interface is native code rather than markup, and cannot be developed or previewed outside a host. That is accepted.
