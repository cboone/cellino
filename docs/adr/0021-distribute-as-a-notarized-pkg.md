# 0021. Distribute as one signed, notarized, stapled pkg

**Status:** Accepted

## Context

The plugin is authored once as a CLAP and projected outward with clap-wrapper ([ADR 0003](./0003-author-clap-project-outward.md)), so a release is not one file. It is a `.clap` that belongs in `Audio/Plug-Ins/CLAP` and a `.component` that belongs in `Audio/Plug-Ins/Components`: two formats, two directories, neither of which a user has any reason to know about.

A macOS download that has not been notarized is refused by Gatekeeper with a dialog that does not distinguish "unnotarized" from "damaged", so notarization is the difference between a download that opens and one that appears broken. `notarytool` accepts a zip, a dmg or a pkg, not a bare bundle, and of those only a pkg can place both bundles itself and be stapled as a single file.

fosforo weighed the three containers and decided this in [its ADR 0014](https://github.com/cboone/fosforo/blob/1317e2b752f7f7d44db9bb7745200ca6e59dde83/docs/adr/0014-distribute-as-a-notarized-pkg.md). Savera ships the same two formats to the same two directories, so the reasoning carries over unchanged, and this ADR adopts it.

## Decision

Ship one signed, notarized, stapled `.pkg` carrying both bundles. Build, sign and notarize it locally, never in CI. The version stays `0.0.0` until v0.1.0.

## Consequences

**The user gets one download and no instructions.**

**The CLAP in the package is the Zig-built one.** The CMake build produces a second CLAP so CI can prove the clap-wrapper seam works, and that one is discarded at packaging time. The `.component` necessarily comes from CMake.

**Signing keys never live in repository secrets.** This repository is public, and hands-off tagged releases would put the Developer ID Application certificate, the Developer ID Installer certificate, both private keys and App Store Connect credentials into CI. That is not a trade worth making for an operation performed rarely and deliberately. A real identity is supplied through `SAVERA_SIGNING_IDENTITY` on the release machine, and `.gitignore` refuses `*.p8`, since an App Store Connect key downloads exactly once.

**The default build stays ad-hoc, and CI asserts it.** Ad-hoc signing is a build step using `/usr/bin/codesign` by absolute path, and `--timestamp --options runtime` are added only when a real identity is supplied. `scripts/assert-adhoc-signature` asserts in CI that the conditional enabling the release path cannot leak into the hermetic build. The positive direction is checked by hand, and the scripts under `scripts/` are the executable record of the procedure.

**The version is changed by hand, in four places, and the installer refuses a mismatch.** `build.zig.zon`, the two plist keys and `packaging/distribution.xml` carry it, and `scripts/build-installer` refuses to package unless all four agree. Cutting v0.1.0 is Phase 7's decision.

**Two certificates are needed, not one.** Bundles are signed with Developer ID Application and the pkg with Developer ID Installer. `security find-identity -v -p codesigning` does not list installer certificates, so a null result there is not evidence that one is missing.

**The release path is not exercised by CI and can rot.** Accepted, as the direct cost of keeping distribution keys out of a public repository.
