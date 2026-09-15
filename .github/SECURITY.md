# Security Policy

## Reporting a Vulnerability

**Please do not report security vulnerabilities through public GitHub issues.**

Instead, please report them through GitHub's private vulnerability reporting:

1. Go to the repository's **Security** tab
1. Click **Report a vulnerability**
1. Fill out the form with details

### What to Include

- Description of the vulnerability
- Steps to reproduce, including the host application and macOS version
- Potential impact
- A saved project, preset or state file that triggers it, if one exists
- Suggested fix, if any

### Response

Savera has a single maintainer, so reports are answered on a best-effort basis, as soon as practical. There is no committed response time.

### What Qualifies as a Security Issue

An audio plugin runs in-process inside a host that holds the user's entitlements, so the threat model is mostly about what hostile or malformed input handed to the plugin can do.

- Memory-safety faults reachable from host-supplied input: audio buffers, events, parameter values, or saved plugin state
- Crashes or corruption triggered by malformed or hostile saved state, or by preset files and preset discovery locations
- Faults reachable from raw MIDI and controller input, including malformed RPN sequences
- Out-of-range or non-finite parameter values, sample rates or block sizes that escape the checks at `activate` and `process` and reach the model
- Path traversal when loading or saving state and presets
- Anything written outside the plugin's expected locations

### Out of Scope

- Issues in upstream dependencies, which should be reported to them directly. This includes CLAP, clap-wrapper and the host applications
- Issues requiring physical access to the machine
- Social engineering attacks
- Crashes only reproducible with a host that violates the CLAP threading contract
