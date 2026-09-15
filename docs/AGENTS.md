# Documentation instructions

- [The build plan](plans/2026-09-13-savera-build-plan.md) permanently stays at the plans root. Phase outcomes record what landed; phase plans move between `plans/todo/` and `plans/done/`. Check that skill invocations do not relocate the permanent plan.
- [ADRs](adr/README.md) own settled decisions and component model contracts. Read the relevant ADR before implementation; supersede it with an amendment or a new ADR when changing a decision.
- Preserve `design/` verbatim, including spelling. Its formatting/lint exclusions are intentional; typos exceptions belong in `../typos.toml` with reasons.
- [Notes](notes/README.md) are living documents. Completed plans are historical records whose figures, citations and line references must not be repaired.
- Read [linters](notes/linters.md) before running or configuring text tools, [CI workflows](notes/ci-workflows.md) before changing CI or interpreting check results, and [skill deviations](notes/skill-deviations.md) before substituting manually for an installed skill.
- Search old measured values across current repository surfaces when correcting them. Keep status in plan outcomes/CHANGELOG and operational depth in notes.
- Root instructions carry critical constraints and navigation. Keep each scoped `AGENTS.md` paired with a `CLAUDE.md` symlink and measure every complete global/root/nested chain against 32 KiB, after formatting too.
