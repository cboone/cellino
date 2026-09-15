# Architecture decision records

Each file records one decision, the context that forced it, and its consequences. Decisions here are settled and should be built on rather than reopened. If one turns out to be wrong, add an amendment section at the foot of the file, or a new ADR superseding it, rather than editing the original text.

Background for all of these lives in [the build plan](../plans/2026-09-13-savera-build-plan.md), which carries the findings, the phase gates and the verification program. [The brainstorm](../design/peti-physical-model-brainstorm.md) is the earlier map of territory the build plan grew from; it is preserved verbatim, and several of its physics claims are corrected by the build plan's findings rather than by editing it.

All twenty-three were locked in the planning pass on 2026-09-13, before any code. 0001 through 0008 are the decisions Savera shares with its siblings [fosforo](https://github.com/cboone/fosforo) and [springer](https://github.com/cboone/springer), restated with Savera's context. 0016, 0021 and 0022 adopt fosforo's 0013, 0014 and 0018 the same way. The rest are Savera's own. Several record a method whose value a later phase measures, such as the solver's iteration count or the internal sample rate; the measured value arrives as an amendment to the ADR that decided the method. Later numbers will record decisions a phase forces once it is under way.

| ADR                                                                  | Decision                                                                       | Status   |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------ | -------- |
| [0001](./0001-macos-on-apple-silicon-only.md)                        | macOS on Apple Silicon only                                                    | Accepted |
| [0002](./0002-zig-pinned-to-0-16-0.md)                               | Zig, pinned to 0.16.0                                                          | Accepted |
| [0003](./0003-author-clap-project-outward.md)                        | Author once as a CLAP instrument; clap-wrapper projects it to an AUv2 `aumu`   | Accepted |
| [0004](./0004-clap-bindings-via-translate-c.md)                      | CLAP bindings from translate-c over a preprocessing step                       | Accepted |
| [0005](./0005-a-pure-model-core-behind-a-seam.md)                    | The model core is pure and names no CLAP type                                  | Accepted |
| [0006](./0006-reject-webview-ui.md)                                  | No WebView UI                                                                  | Accepted |
| [0007](./0007-no-allocation-on-the-audio-thread.md)                  | Nothing on the audio thread allocates, locks, or makes a syscall               | Accepted |
| [0008](./0008-parameters-identified-by-stable-clap-id.md)            | Parameters are identified by stable `clap_id`                                  | Accepted |
