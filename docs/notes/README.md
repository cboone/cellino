# Notes

Each file here records how some part of this project actually behaves: what was measured, what was refused, and which instrument answers which question. They exist because [`AGENTS.md`](../../AGENTS.md) is loaded into an agent's context in full on every session and a reference document is not, so depth that is only needed while working on one area belongs here, and a rule that prevents damage belongs there.

**A note is not an ADR and not a plan, and the difference decides how it is edited.**

- A **note** records how something currently behaves. It is a living document: when a measurement here goes stale, the repair is a fresh measurement written in place, and a `path:line` citation in one is a claim about current code.
- An [**ADR**](../adr/README.md) records a decision. It is settled, and it is superseded by an amendment or a new ADR, never rewritten.
- A **plan** in [`docs/plans/done/`](../plans/done/) is a historical record of what was intended at the time, and it is never corrected at all. [The build plan](../plans/2026-09-13-savera-build-plan.md) is the one permanent plan; it holds the sequencing, the phase gates and the verification program, and it never moves.

**Every figure here was measured rather than reasoned about**, and each carries the date it was measured. Before changing one, grep for the old value across the repository: figures here are quoted in `AGENTS.md`, the ADRs, the build plan, and sometimes a workflow or config comment, so a correction applied in one place leaves the stale copies reading as current. Occurrences in `docs/plans/done/` are not live and are left alone.

| Note                                      | Read it before                                                                                              |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| [CI workflows](./ci-workflows.md)         | editing anything under `.github/workflows/` or `.github/dependabot.yml`, or reading a pull request's checks |
| [Linters](./linters.md)                   | running or configuring Prettier, markdownlint, typos, actionlint or gitleaks                                |
| [Skill deviations](./skill-deviations.md) | doing by hand something a catalog skill should do, or writing one of those skills from this project         |
