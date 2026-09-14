# Savera Phase 0: repository foundation

## Context

[The build plan](../2026-09-13-savera-build-plan.md#phases) scopes Phase 0 as the repository foundation: `AGENTS.md` recording that the build plan never leaves `docs/plans/`, the `CLAUDE.md` symlink, `.claude/` seeded with the Zig permissions, `.github/` workflows and `*.instructions.md`, lint configuration, `docs/{adr,design,notes,plans}` with a notes index and a skill deviation record, the twenty-three ADRs, and the community files. Its gate is CI green on that scaffolding (markdownlint, Prettier, typos, actionlint, gitleaks, TruffleHog, and a shell job that passes by finding nothing until Phase 1), every relative link resolving, the preserved brainstorm untouched with the negative control that Prettier run with its ignore file bypassed does flag it, and the deviation record carrying an entry for each catalog issue the phase re-derived by hand.

The repository holds six tracked files today: a three-line `.gitignore`, the MIT `LICENSE`, a one-line `README.md`, the brainstorm at `docs/design/peti-physical-model-brainstorm.md`, the master build plan at `docs/plans/2026-09-13-savera-build-plan.md`, and the completed rename plan in `docs/plans/done/`. `feature/phase-0` sits at `dea180f`, the same commit as `main`. Nothing in Phase 0 is Zig: a Zig CI workflow reads its toolchain from `build.zig.zon`, so `ci.yml` arrives with that file in Phase 1 rather than ending Phase 0 red.

[springer's Phase 0 plan](https://github.com/cboone/springer/blob/main/docs/plans/done/2026-09-08-springer-phase-0-repository-foundation.md) is the direct precedent. [fosforo](https://github.com/cboone/fosforo) is the precedent for what that repository has learned since springer copied it: Prettier owning formatting with markdownlint verifying (fosforo #85 and #106), `AGENTS.md` as a hub over topic notes (fosforo #117), caller jobs named for their tool, and Dependabot over action and npm pins. Most of Phase 0 is adaptation of those two repositories rather than invention.

This repository is not a fork: `origin` is `cboone/savera`, there is no `upstream` remote, and `gh repo view` reports it public, not archived, and not a fork. Plans stay committed here, per the `cboone` convention.

## Decisions

Settled with the owner on 2026-09-14, before any file was written.

| Question                                  | Decision                                                                                                                                                                                                                                                       |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| How the text lint job runs                | A call to `cboone/gh-actions` `lint-text.yml` at v3.2.0 with `use-consumer-versions: true`. That job runs `prettier --check .`, so Prettier formats YAML, JSON and JSONC as well as Markdown, and markdownlint verifies the Markdown.                          |
| Which by-hand gates become CI checks      | Relative links only, through `markdownlint-rule-relative-links` as a `customRules` entry. The brainstorm's byte identity and `AGENTS.md`'s size stay by-hand checks at each phase pull request.                                                                |
| How the typos word list is built          | A word joins `typos.toml` only when typos flags it, with its reason as a comment. There is no pre-seeded organological or Hindustani vocabulary; the list is whatever the prose actually trips.                                                                |
| What `.claude/settings.json` allows       | `zig build` and `zig fmt`, read-only GitHub commands, and the read-only forms of the Phase 0 lint tools. Nothing in the lint set can write.                                                                                                                    |
| The shape of `AGENTS.md`                  | A hub from the first real version: overview, current state, non-negotiables, identifiers, a short Rules list, and a pointer table into `docs/notes/`. Tool traps live in `docs/notes/linters.md` and `docs/notes/ci-workflows.md`, not in a flat gotchas list. |
| Pull request titles                       | Sentence case with no type prefix, stated in `CONTRIBUTING.md`. Commit subjects stay Conventional Commits.                                                                                                                                                     |
| Milestones and issues for Phases 1 and 2  | Not filed by Phase 0. Each is filed as the first step of planning its phase, against what Phase 0 actually landed.                                                                                                                                             |
| How the master plan absorbs moved facts   | Phase 0 appends its outcomes subsection and updates the Status line of the one Risks entry that moved. The dated Findings and Environment status tables stay as they were measured.                                                                            |
| The GitHub repository description         | "A modulatable synthesizer built on a physical model of the Indian hand harmonium (peti), for macOS (CLAP and AUv2)"                                                                                                                                           |
| Licensing                                 | The existing MIT `LICENSE` only. Whether to adopt REUSE is decided when the first CC0 fixture lands in Phase 2.                                                                                                                                                |
| The security policy's response commitment | Private vulnerability reporting, answered on a best-effort basis by a single maintainer, with no hour commitments.                                                                                                                                             |

## Findings that shaped the plan

Measured on 2026-09-14 in this worktree unless stated otherwise.

### gh-actions moved twice since the build plan measured it

The build plan's environment table records `cboone/gh-actions` at v3.1.1, with `set-up-clap-validator` on `main` only. v3.2.0 was released on 2026-09-14 at `0d53592f40b487f01b26b374e539c517fa9c570f`, and three of its changes bear on Phase 0 and Phase 1:

- **`set-up-clap-validator` is released.** The Risks entry "gh-actions release timing" is resolved, and Phase 1 consumes the action from a release rather than copying fosforo's composite action.
- **`lint-github-actions.yml` installs a pinned, checksum-verified shellcheck** and reports both tool versions before running actionlint (gh-actions #85). Without shellcheck on `PATH`, actionlint skips every `run:` block and still exits 0. The version report is the positive control, and Phase 0 reads it from a real run rather than assuming it.
- **`install-pinned-tool` is a composite action** (gh-actions #87) that installs a release binary pinned by version and SHA-256. It replaces springer's hand-written download, `sha256sum` and `tar` steps for typos, which no reusable workflow runs.

v3.1.1 had already fixed `lint-text.yml` (gh-actions #83), the defect that made springer inline its text lint job. The `use-consumer-versions` input installs the tools from this repository's own `package-lock.json` with `npm ci`, so local runs and CI resolve identical versions with per-package integrity.

`lint-shell.yml` at v3.2.0 still discovers extensionless scripts only under `bin/`, `scripts/` and `script/`, which is tracked upstream as gh-actions #86. Phase 0 adds no shell script, so the job passes by finding nothing, as the build plan states. Shebang-based discovery stays in Phase 1, where the first scripts under `cmake/` arrive.

### Catalog issue #345 is closed, and its skill is installed

The build plan's finding row says the catalog's Zig and audio skills are unwritten. `plant-defects` (#345) has since shipped and is installed at 1.0.0. Phase 0 uses it for its controls rather than re-deriving the plant table, and the deviation record says so. Issues #339, #340, #341, #342, #343, #344, #346 and #347 remain open.

### The scaffolding skills are unchanged since springer measured them

- `bootstrap-project` is still the v1.3.1 cache copy whose execution step reads `plugins/*/commands/*.md` files that exist nowhere. The marketplace checkout's `SKILL.md` no longer references `commands/` but still declares 1.3.1, so the fix appears to have merged without a version bump, and an installed cache has no reason to refresh. No issue tracks that. Phase 0 invokes the individual skills in the orchestrator's documented order instead, and files the version defect in step 0.10.
- `scaffold-new-repo` 1.8.3's `zig-cli` type selects a `.gitignore` template and a README snippet and nothing else. It asks before overwriting an existing `README.md` or `LICENSE`, and it ends with a `git add -A` commit.
- `set-up-linters` 2.1.0 offers typos only on its Rust path (tracked as #353), recommends cspell, and writes a lint workflow template whose `paths-ignore` skips `*.md` and `docs/**`, which is the opposite of this repository's rule. Its closing step offers `lint-and-fix`, whose markdownlint fixer the build plan forbids.
- `set-up-secret-scanning` 3.0.2 names the caller job `scan` in both workflows. fosforo renamed its jobs after two identical `scan / Validate inputs` rows appeared in a pull request's check rollup. The templates pin gh-actions v3.0.0, and the skill itself says to refresh that pin when emitting.

### Baselines, at the versions CI will pin

The local Homebrew Node tools are not the versions CI will run: Prettier 3.8.1 and markdownlint-cli2 0.21.0 locally, against Prettier 3.9.6 and markdownlint-cli2 0.23.2 current on npm. The Prettier, markdownlint and relative-links figures below come from those pinned versions installed in a scratch directory and run over a `git archive` copy of `HEAD`. typos and gitleaks ran from Homebrew, whose versions already match what CI will pin: typos 1.50.1, and gitleaks 8.30.1, the default of gh-actions v3.2.0's `scan-for-secrets.yml`.

| Check                                                                                                               | Result                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `typos` over the whole tree, at 1.50.1                                                                              | 0 findings                                                                                                                |
| `gitleaks detect`, at 8.30.1                                                                                        | No leaks across 7 commits                                                                                                 |
| `prettier --check "**/*.md"`, `docs/design/` ignored                                                                | Clean                                                                                                                     |
| Prettier over the brainstorm with `--ignore-path /dev/null`                                                         | Flagged, exit 1. The negative control works                                                                               |
| markdownlint-cli2 with fosforo's rules, `docs/design/**` ignored                                                    | 0 issues in 3 files                                                                                                       |
| The same rules over the brainstorm with the ignore removed                                                          | 13 issues: `MD034` 8, `MD036` 4, `MD040` 1                                                                                |
| `prettier --check .` over a real `package-lock.json`, springer's workflows, fosforo's `dependabot.yml`, JSON, JSONC | Clean. Prettier's YAML and JSON output already matches all of them                                                        |
| `markdownlint-rule-relative-links` 5.1.2 over the tree                                                              | 0 issues                                                                                                                  |
| The same rule over a planted file                                                                                   | Reports a missing file and a bad `#fragment` as separate findings, and accepts a valid fragment and a directory link      |
| The typos 1.50.1 `x86_64-unknown-linux-musl` tarball                                                                | SHA-256 `edf0545109aee6a22751d04ddecb97c45be47d3aa0409564fb895eeeace91b1e`, identical to springer's pin; binary `./typos` |

The interview assumed that `prettier --check .` would flag `package-lock.json`, and the measurement contradicts it. The lockfile is listed in `.prettierignore` anyway, for a different reason: npm writes those bytes, so no formatter should be judging them. The comment beside the entry says it is not flagged today.

The brainstorm's SHA-256 is `ee012eff0fa93b1ae73dc311e516e1c709ab30b15319bfbfcd0f1bb91ce13529`, last changed by `b271635`, the rename. Phase 0 must leave that digest unchanged.

### The `commit` and `pr` skills would move the master plan

Both skills move a plan from the `docs/plans/` root to `done/` once they judge its work complete, and neither implements the `lifecycle: permanent` exemption catalog #347 asks for. The Phase 0 pull request is the first in this repository to edit the master plan, which is exactly the case the build plan's Context warns about. `AGENTS.md` states the rule, and step 0.17 verifies that the master plan did not move.

### Repository settings

Private vulnerability reporting is enabled. Discussions are disabled, so `CONTRIBUTING.md` points at the issue tracker only. `main` has no branch protection, and required checks are not part of Phase 0. No bulk reformat happens here, so `.git-blame-ignore-revs` is not needed.

## Work

Every commit is GPG-signed with a Conventional Commits subject. A skill named below is invoked through the Skill tool rather than replicated by hand; where this plan adapts what a skill writes, the adaptation lands in the same commit as the skill's output. Invoke `write-markdown` before writing any Markdown file.

| Step | Work                                                                                          | Skills                                         | Commit                                                                         |
| ---- | --------------------------------------------------------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------ |
| 0.0  | This plan                                                                                     | `write-markdown`                               | `docs: plan Phase 0, the repository foundation`                                |
| 0.1  | Agent config stub, `CLAUDE.md` symlink, `.claude/settings.json`, `CHANGELOG.md`, ignore rules | `scaffold-new-repo`, scoped                    | `chore: scaffold agent config, changelog and ignore rules`                     |
| 0.2  | `gitleaks.yml`, `trufflehog.yml`, `.gitleaks.toml`                                            | `set-up-secret-scanning`                       | `ci: scan for secrets with gitleaks and TruffleHog`                            |
| 0.3  | Pinned lint tools and their configuration                                                     | `set-up-linters`, then adapted                 | `chore: pin and configure the text lint tools`                                 |
| 0.4  | `.github/workflows/text-lint.yml`                                                             | none; reusable workflows                       | `ci: lint text, shell, workflows and spelling`                                 |
| 0.5  | `.github/dependabot.yml`                                                                      | none; adapted from fosforo                     | `ci: track action and npm pins with Dependabot`                                |
| 0.6  | `docs/adr/README.md` and ADRs 0001 to 0008                                                    | `write-markdown`                               | `docs: add ADRs 0001 to 0008, the family's shared decisions`                   |
| 0.7  | ADRs 0009 to 0015                                                                             | `write-markdown`                               | `docs: add ADRs 0009 to 0015, the instrument model`                            |
| 0.8  | ADRs 0016, 0021 and 0022                                                                      | `write-markdown`                               | `docs: add ADRs 0016, 0021 and 0022, adopted from fosforo`                     |
| 0.9  | ADRs 0017 to 0020 and 0023                                                                    | `write-markdown`                               | `docs: add ADRs 0017 to 0020 and 0023, the plugin surface`                     |
| 0.10 | `docs/notes/`: index, linters, CI workflows, skill deviations; file any new skill defects     | `write-markdown`, `create-issue`               | `docs: add the notes index, the linter and CI notes, and the deviation record` |
| 0.11 | `AGENTS.md` as the hub                                                                        | `write-markdown`                               | `docs: write AGENTS.md as the hub document`                                    |
| 0.12 | `copilot-instructions.md` and the three `*.instructions.md` files                             | `write-markdown`, then `clean-up-agent-config` | `docs: add code review instructions`                                           |
| 0.13 | `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, pull request template                 | `write-markdown`                               | `docs: add community files`                                                    |
| 0.14 | `README.md` rewritten with status and roadmap; repository description set                     | `write-markdown`                               | `docs: rewrite the README with the status and roadmap`                         |
| 0.15 | The local gate and the local plants                                                           | `plant-defects`, `pin-everything` as an audit  | none; results recorded in 0.17                                                 |
| 0.16 | Review, push, open the pull request, the CI plant, monitor                                    | `review-branch`, `pr`, `monitor-pr`            | `test: plant one defect for each CI lint instrument`, then its revert          |
| 0.17 | Outcomes in the master plan, the Risks status, this plan's results, the move to `done/`       | `commit`                                       | `docs: record Phase 0 outcomes`, then `docs: move the Phase 0 plan to done`    |

### 0.0 This plan

Commit this file alone, before any other change, so each later commit can be read against it.

### 0.1 Agent config, changelog and ignore rules

Invoke `scaffold-new-repo` with project name `savera`, project type `zig-cli`, and the description from Decisions. Scope it down:

- **Do not overwrite `README.md` or `LICENSE`.** The licence is already correct (MIT, 2026, Christopher Boone), and the README is rewritten deliberately in 0.14.
- Take `AGENTS.md` (a stub, replaced in 0.11), `CLAUDE.md` as a relative symlink to it, `.claude/settings.json`, `.github/copilot-instructions.md` (replaced in 0.12), `CHANGELOG.md`, and the `.gitignore` merge.
- Take `docs/plans/todo/.gitkeep`, because `todo/` is left empty when this plan moves in 0.17 and later phase plans land there. Skip `docs/plans/done/.gitkeep`, since `done/` already holds a plan.
- Skip the skill's own `git add -A` commit. This step commits exactly its own files.

Fill the `allow` list in `.claude/settings.json` from Decisions. The lint entries are exact commands rather than prefixes, because a prefix such as `Bash(typos *)` would also allow `typos --write-changes`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(zig build *)", "Bash(zig fmt *)", "Bash(gh pr view *)", "Bash(gh pr checks *)", "Bash(gh run view *)", "Bash(gh issue view *)", "Bash(npm run format:check)", "Bash(npm run lint:md)", "Bash(typos)", "Bash(typos --format brief)", "Bash(actionlint)", "Bash(gitleaks detect *)", "Bash(shellcheck *)"],
    "deny": []
  }
}
```

`npm ci`, `npm run format` and `prettier --write` are deliberately absent, since each writes. Prettier sets the final layout of this file in 0.3.

Extend the merged `.gitignore` past the skill's Zig template with the entries fosforo and the build plan need, each with its comment:

- `zig-pkg/`, `build/` for the CMake output, and `dist/` for release artifacts.
- `*.p8`, since an App Store Connect key downloads exactly once and the copy on disk is the only copy.
- `node_modules/`, noting that `package.json` and `package-lock.json` are tracked because the lockfile is the pin.
- `verification/*` with `!verification/.gitkeep`, and a tracked `verification/.gitkeep`. The build plan puts captures, purchased audio and renders there, and purchased libraries must never reach a public history. fosforo had to rewrite three commits to remove a single capture committed before its equivalent rule existed.

Python entries wait for Phase 2's harness, and `fixtures/` is committed by design.

`CHANGELOG.md` is Keep a Changelog 1.1.0 with an empty `## [Unreleased]`. Phase 0 makes no user-facing change, so nothing goes under it.

### 0.2 Secret scanning

Invoke `set-up-secret-scanning` with `both`, and accept its `.gitleaks.toml`. Then adapt:

- **Refresh both pins** to `0d53592f40b487f01b26b374e539c517fa9c570f # v3.2.0`, re-running the skill's own `gh release view` and `gh api` refresh at authoring time in case a newer release has been cut.
- **Name the caller jobs `gitleaks` and `trufflehog`** rather than `scan`, with fosforo's comment explaining that two workflows' identical `scan / Validate inputs` rows make an ambiguous rollup and branch protection target.
- **Pass `timeout-minutes: 3`** as an input, with the reason it is an input rather than a job key. The figure is carried from fosforo (24 seconds maximum over 29 gitleaks runs, 53 seconds over 28 TruffleHog runs) and is not yet measured here; v3.2.0 now applies it to the `Validate inputs` job as well (gh-actions #81).
- **No `paths-ignore`**, with the comment that a secret can be committed into any file.

Replace the starter allowlist's lockfile entries, none of which apply, with these, each commented:

```toml
[allowlist]
paths = [
  '''build\.zig\.zon''',
  '''^build/''',
  '''^zig-out/''',
  '''^zig-pkg/''',
  '''^fixtures/.*\.(wav|npz)$''',
]
```

The first four are fosforo's: Zig pins dependencies by multihash, which trips the generic entropy rules and is deliberately public, and the build directories hold fetched third-party trees. The last is the fixture entropy pattern the build plan names, on audio-tools' precedent for its 32-bit float probes: committed CC0 cuts and rendered NumPy vectors are high-entropy bytes that mean nothing. `package-lock.json` is scanned like any other file, as audio-tools scans its own. Neither `build.zig.zon` nor `fixtures/` exists yet; the entries are written now because this file is authored once and the reasons are known, and plants P10 and P11 test the fixture entry rather than trusting its regex.

### 0.3 Lint tools and configuration

Invoke `set-up-linters` after 0.2, since it offers actionlint only once `.github/workflows/` exists. Take Prettier, EditorConfig, markdownlint-cli2 and actionlint. Decline cspell, because typos covers the same ground and two dictionaries is two places to add every word. Decline its CI step, because 0.4 writes the workflow without the template's `paths-ignore`. Decline its offer to run `lint-and-fix`, for the reason below.

Then write or adapt each file:

- **`package.json`**, named `savera-text-lint-tools`, `private: true`, `engines.node` `>=22`, and a description saying it pins the text tools and that `zig build` never reads it. `devDependencies` are exact, with no caret or tilde: `markdownlint-cli2` 0.23.2, `markdownlint-rule-relative-links` 5.1.2, `prettier` 3.9.6. The scripts spell out exactly the commands `lint-text.yml` runs, so that CI and the documented local invocation are one string: `format` is `prettier --write .`, `format:check` is `prettier --check .`, and `lint:md` is `markdownlint-cli2 "**/*.md"`. Generate `package-lock.json` with `npm install`, then confirm `npm ci` reproduces it.
- **`.npmrc`** with `engine-strict=true` and springer's comment. Re-measure the Node floor with `npm view <package>@<version> engines` for all three packages, and record the highest in the comment and in `engines`.
- **`.prettierrc.json`**: `printWidth: 10000`, `proseWrap: "preserve"`, and `embeddedLanguageFormatting: "off"`. The last is fosforo's #106 lesson: Prettier dedented six YAML fences inside Markdown files before it was set.
- **`.prettierignore`**, each entry commented: `docs/design/`, kept in step by hand with the markdownlint ignore; `package-lock.json`, for the ownership reason in Findings; `node_modules/`, listed for the reader although Prettier skips it unprompted; `build/`, `zig-out/`, `zig-pkg/`, `.zig-cache/`, `dist/` and `verification/`, none of which is authored.
- **`.markdownlint-cli2.jsonc`**, fosforo's with each comment adapted: `MD013` off, `MD024` siblings only, `MD033` off, `MD049` off, `MD060` aligned. Add `"customRules": ["markdownlint-rule-relative-links"]` with a comment saying what it checks (a linked file exists, and a `#fragment` names a heading in it) and that it is the gate's relative-link check. `globs` is `["**/*.md"]`. `ignores` lists `node_modules/**` (load-bearing, since markdownlint-cli2 walks into it and the CI job runs `npm ci` in the workspace), `build/**`, `zig-out/**`, `zig-pkg/**`, `.zig-cache/**`, `dist/**`, `verification/**`, and `docs/design/**`.
- **`.editorconfig`**, fosforo's sections: `[*]`, `[*.{zig,zon}]`, `[*.{c,cpp,h}]`, `[CMakeLists.txt]`, `[*.{json,jsonc,toml,yml,yaml}]` and `[*.md]`. End it with a comment that the shell section arrives with Phase 1's first script, carrying shfmt's all-or-nothing reading of this file, and that a Python section arrives with Phase 2's harness.
- **`typos.toml`**, adapted from fosforo and written by hand, since no skill writes it outside Rust. Keep its two `extend-ignore-re` entries with their reasons: the backtick-anchored `[0-9a-f]{7,40}` short-SHA pattern, and the `spellchecker:off` and `spellchecker:on` span. `[files] extend-exclude` lists `zig-out/`, `zig-pkg/`, `build/`, `.zig-cache/`, `node_modules/` and `verification/`. There are no `extend-words` or `extend-identifiers` entries until typos flags something. After `package-lock.json` exists, run `typos` again; if it flags anything in the lockfile, exclude the file with the finding recorded as the reason.

Then run `npm ci`, `npm run format`, `npm run format:check`, `npm run lint:md`, `typos` and `gitleaks detect --no-banner`, and confirm the tree is clean. The baseline predicts Prettier changes nothing. If it changes a file, commit that separately as `style: format <files> with prettier` before this step's commit, so the configuration commit carries no reformat.

**`lint-and-fix` is not used in this repository.** It passes each detected linter its fix flag, and `markdownlint-cli2 --fix` ignores its file arguments and rewrites every file its globs match, which the build plan forbids. `write-markdown`'s validation step asks for the linter in fix mode for the same reason and is satisfied here the same way: `npm run format` fixes, and `npm run lint:md` verifies. `docs/notes/linters.md` records the substitution.

### 0.4 The lint workflow

One file, `.github/workflows/text-lint.yml`, with four caller jobs. Three call reusable workflows; `typos` is its own job, because no reusable workflow runs typos.

````yaml
name: text-lint

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  text:
    uses: cboone/gh-actions/.github/workflows/lint-text.yml@0d53592f40b487f01b26b374e539c517fa9c570f # v3.2.0
    with:
      use-consumer-versions: true
      run-markdownlint: true
      run-prettier: true
      run-cspell: false
      run-yamllint: false
      timeout-minutes: 5

  shell:
    uses: cboone/gh-actions/.github/workflows/lint-shell.yml@0d53592f40b487f01b26b374e539c517fa9c570f # v3.2.0
    with:
      timeout-minutes: 3

  actions:
    uses: cboone/gh-actions/.github/workflows/lint-github-actions.yml@0d53592f40b487f01b26b374e539c517fa9c570f # v3.2.0
    with:
      timeout-minutes: 3

  typos:
    runs-on: ubuntu-latest
    timeout-minutes: 3
    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1

      - uses: cboone/gh-actions/actions/install-pinned-tool@0d53592f40b487f01b26b374e539c517fa9c570f # v3.2.0
        with:
          tool: typos
          version: "1.50.1"
          url-template: https://github.com/crate-ci/typos/releases/download/v{version}/typos-v{version}-x86_64-unknown-linux-musl.tar.gz
          checksum: edf0545109aee6a22751d04ddecb97c45be47d3aa0409564fb895eeeace91b1e
          archive-member: ./typos

      - name: Report tool version
        run: typos --version

      - name: Check spelling
        id: typos
        run: typos

      - name: Summarize the findings
        if: failure() && steps.typos.outcome == 'failure'
        run: |
          {
            echo "### typos"
            echo
            echo '```text'
            typos --format brief || true
            echo '```'
          } >> "${GITHUB_STEP_SUMMARY}"
````

The comments carry the reasoning, adapted from fosforo and springer:

- **No `paths-ignore`.** A check must not be able to skip the change that governs it: `typos.toml`, `.markdownlint-cli2.jsonc`, `.prettierrc.json`, `.prettierignore` and this file all live in the set these jobs read, and nearly every Phase 0 change is a Markdown change. `ci.yml` will carry `paths-ignore` from Phase 1, correctly, because it is a set of macOS builds.
- **A bare `pull_request:`** with no `branches:` filter, because that filter matches the base ref and silently skips a stacked pull request (fosforo's #87 trigger finding).
- **Caller jobs are named for what they check**, so the rollup reads `text / Text lint`, `shell / Shell lint`, `actions / actionlint` and `typos`.
- **`use-consumer-versions: true`** makes the committed lockfile the pin, which is also what lets the `relative-links` custom rule resolve from the workspace's `node_modules`.
- **The timeouts are carried rather than measured**: 3 is the floor fosforo's Ubuntu static-analysis jobs share, and `text` gets 5 for its `npm ci` of the whole tool tree. Re-measure both from this repository's first runs and record the figures in `docs/notes/ci-workflows.md`.
- **yamllint is off.** It is not part of the gate, Prettier owns YAML formatting, and actionlint owns workflow semantics.
- **typos is pinned by version and SHA-256**, since a release asset can be replaced under the same tag, and through `install-pinned-tool` rather than inline shell. Dependabot moves the action's SHA but cannot see `version` or `checksum`, which `upgrade-everything` sweeps.

Refresh `actions/checkout` to its current release SHA at authoring time. Before committing, confirm `shellcheck --version` prints, then run `actionlint`; without shellcheck on `PATH`, actionlint's silence means nothing (plant P9).

### 0.5 Dependabot

`.github/dependabot.yml`, adapted from fosforo: the `github-actions` ecosystem with `directory: /` (this repository has no composite actions of its own yet), and the `npm` ecosystem with `versioning-strategy: increase` so that exact pins stay exact. Both run weekly on Monday at 09:00 America/New_York, group minor and patch updates, open majors individually, use the `chore(deps)` commit prefix, and label `dependencies`. The header comment names what Dependabot cannot see: the typos `version` and `checksum` inputs, and from Phase 1 anything pinned in `build.zig.zon`.

Dependabot reads its configuration only from the default branch, so its first run is a post-merge check, listed under Follow-ups.

### 0.6 to 0.9 The twenty-three ADRs

The format is fosforo's, which springer adopted: `# NNNN. Title in sentence case`, then `**Status:** Accepted`, then `## Context`, `## Decision` and `## Consequences`. Filenames are `NNNN-kebab-case.md`, zero-padded and never renumbered, with dots in versions becoming hyphens. A changed decision is recorded by appending a `## Amended by issue #N: what changed` section, never by editing the original text.

Rules for all twenty-three:

- **The build plan's table row is the Decision sentence, not the file.** Context says what forced the decision; Consequences names what is given up, what is recorded as a deficit, and which phase measures or proves it. springer's planning-pass ADRs run 25 to 40 lines, and that is the density to write to.
- **Cite the build plan by section link**, such as `../plans/2026-09-13-savera-build-plan.md#the-solver`, so the `relative-links` rule verifies the anchor as well as the file.
- **A value the build plan leaves to measurement stays open** and names its phase: `K`, the internal rate, the jet crossover, the pressure-to-pitch coefficient, the near-field pipe placement. The measured value arrives as an amendment.
- **An ADR that names a check says which plant proves it**, as springer's ADR 0005 does.
- **0001 to 0008 start from springer's text** and restate savera's context. **0016, 0021 and 0022 adopt fosforo's 0013, 0014 and 0018**, linking to them by permalink at fosforo's `HEAD` commit at authoring time, since fosforo amends its ADRs.
- **Run `typos` after each group.** The ADRs introduce most of the project's vocabulary, and a word is added only when flagged.

`docs/adr/README.md` lands in 0.6 and gains its rows with each group. It adapts fosforo's framing: each file records one settled decision, superseded rather than edited; background is in the build plan, and the brainstorm is the earlier map that the build plan's findings correct in places; all twenty-three were locked in the planning pass on 2026-09-13, before any code; 0001 to 0008 are shared with fosforo and springer, 0016, 0021 and 0022 adopt fosforo's, and later numbers will record decisions a phase forces.

| ADR  | File                                                         | Decision                                                                                                           | Draws on                                                                 | Must also record                                                                                                                                                                                                                                                                                                                                        |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0001 | `0001-macos-on-apple-silicon-only.md`                        | macOS on Apple Silicon only                                                                                        | springer 0001; Build architecture                                        | The model, render harness and unit tests build on Linux so the oracle can run beside the Ubuntu `python` job. That is a CI convenience, not a portability promise. The macOS 11.0 deployment target is stated in four files that must agree.                                                                                                            |
| 0002 | `0002-zig-pinned-to-0-16-0.md`                               | Zig, pinned to 0.16.0                                                                                              | springer 0002; Environment status; Build architecture; Risks             | `build.zig.zon` is the single source and CI reads it. The object-file workaround for 0.16's archive alignment is tied to this pin. 0.17's known hazards (runtime vector indexing, the float-mode builtin, translate-c's backend) and the migration-safe idioms.                                                                                         |
| 0003 | `0003-author-clap-project-outward.md`                        | Author once as a CLAP instrument; clap-wrapper projects it to an `aumu` Audio Unit                                 | springer 0003; the `aumu`, tail and preset findings; Identifiers         | Passing `AUV2_INSTRUMENT_TYPE "aumu"` makes clap-wrapper's `features[0]` mapping unreachable, so the controls are a unit test on `features[0]` and a `plutil` read of the built plist, each proven by plant, and never a grep for the wrapper's warning. The clap-wrapper pin and its three PRs. `auval` expected to see nothing.                       |
| 0004 | `0004-clap-bindings-via-translate-c.md`                      | CLAP bindings from `translate-c` over a `zig cc -E -P` preprocessing step                                          | springer 0004; Build architecture                                        | Restated macros, `CLAP_PLUGIN_FEATURE_INSTRUMENT` among them. Comptime layout assertions over every ABI-crossing struct. The configure-time check that the CLAP headers clap-wrapper downloads match the tag `build.zig.zon` pins.                                                                                                                      |
| 0005 | `0005-a-pure-model-core-behind-a-seam.md`                    | The model core is pure and names no CLAP type                                                                      | springer 0005; Source layout                                             | Three layers. The innermost may name a sample rate and may not name a sample offset or a note id. Enforcement is a run-time walk of `src/model/` from the source root, not a comptime assertion, because the siblings' comptime pattern pins signatures rather than imports. The Phase 2 plant.                                                         |
| 0006 | `0006-reject-webview-ui.md`                                  | No WebView UI                                                                                                      | springer 0006                                                            | It binds whatever GUI ADR 0020 eventually admits.                                                                                                                                                                                                                                                                                                       |
| 0007 | `0007-no-allocation-on-the-audio-thread.md`                  | Nothing reachable from the audio thread allocates, locks, or makes a syscall                                       | springer 0007; The solver; The internal rate; Voices, capacity           | Every capacity derived in one place with its derivation. Single-writer relaxed atomics drained on the main thread. Savera starts no thread, a source canary says so, and so no Thread Sanitizer harness exists. The activate-time arena. FPCR flush-to-zero around `process()`. Runtime `if` at trust boundaries, because ReleaseFast removes `assert`. |
| 0008 | `0008-parameters-identified-by-stable-clap-id.md`            | Parameters are identified by stable `clap_id`                                                                      | springer 0008; Parameters and the modulation surface; the rescan finding | Rescan-gated flags and ranges declared before the first release, at the widest range the engine will ever accept. Numbered groups with gaps. `clap.plugin-auv2-param-ordering`, because clap-wrapper otherwise sorts by id. The Range parameter clamps at runtime so unlocking changes no declared bound.                                               |
| 0009 | `0009-single-mode-reed-behind-a-reed-seam.md`                | The reed is a single-mode oscillator with Millot's pumped and jet flows, behind a reed seam                        | The reed; the arXiv 2401.01606, 1 kPa and pressure-to-pitch findings     | The seam exports the two sensitivities the solver's Jacobian needs, so a modal reed or the clamped bar replaces it without touching the rest. The validity ceiling near 1 kPa and the missing attack modes are named deficits. The Q bracket, 95 and about 650. The pressure-to-pitch coefficient is measured in Phase 2.                               |
| 0010 | `0010-fixed-k-bracketed-newton-solver.md`                    | Fixed-K bracketed Newton per reed on the near-field pressure, with the chamber and reservoir updated explicitly    | The solver; the Millot topology finding                                  | Two square roots in one residual and the single-root question. The regularised jet. `K`, the crossover and the worst case open until Phase 2. The overrun counter. The explicit coupling's stability condition, and the Phase 5 check against the bordered-Newton reference, recorded as an amendment. `.strict` float mode.                            |
| 0011 | `0011-fixed-internal-sample-rate.md`                         | The model runs at a fixed internal rate chosen by measurement and is resampled to the host rate                    | The internal rate and the resampler; the 8x oversampling finding         | Why a host-rate multiple would defeat the oracle. The 2, 4, 8 and 16 times 48 kHz study. The rational polyphase resampler and its tabulated ratios. Untabulated host rates refused at `activate`. `clap.latency` and `clap.tail`, and the Audio Unit's missing tail as a wrapper deficit.                                                               |
| 0012 | `0012-reservoir-chambers-and-a-downstream-pallet.md`         | A spring-loaded reservoir feeds sealed bank chambers through stop orifices, with the pallet downstream of the reed | The air path; the pallet and in-rank coupling findings                   | Why air compressibility is not the reservoir (about 130 millilitres at 1 kPa). No in-rank coupling terms, as a recorded refusal. The near-field pipe placement and whether `V1` scales with the bank are Phase 2 findings.                                                                                                                              |
| 0013 | `0013-keys-are-continuous-valves.md`                         | Keys are continuous valves; velocity drives only pallet rate and key noise                                         | Keys, coupler, drones, scale changer, tremolo; the key-depth finding     | Depth comes from note expressions, polyphonic aftertouch, or the note-on. The pitch pull is modelled as an acoustic load, because a reduced pressure difference alone would move the reed the other way. The only measured figure is from an accordion reed.                                                                                            |
| 0014 | `0014-flow-derivative-through-an-enclosure-filter.md`        | The output is the derivative of total reed flow through an enclosure filter estimated from a recording             | The cabinet; the DAFx-23 filter finding                                  | Flow versus its derivative changes the tilt by 6 dB per octave. The jali open and closed are preset dimensions. The monopole approximation is deliberate, and the stereo port carries it on both channels because channel count is rescan-gated.                                                                                                        |
| 0015 | `0015-python-discovers-the-reed-and-freezes-as-oracle.md`    | Python discovers the single reed and is frozen as the oracle for that seam                                         | The harness and the oracle                                               | PEP 723 scripts under uv, with Numba. Everything downstream of the reed seam is Zig only, with the render harness as the shared instrument. The lockstep and perceptual tiers, NaN rejection, and the plant table. The Numba-supported Python pin is scheduled work.                                                                                    |
| 0016 | `0016-harnesses-are-build-steps.md`                          | Harnesses are executables behind their own build steps, never part of `zig build test`                             | fosforo 0013                                                             | Exit codes 0, 1 and 2. Judging scripts whose assertion order tells an absence apart from an instrument that did not run, which is `plant-defects`' ordered assertions. The render harness and the offline host harness.                                                                                                                                 |
| 0017 | `0017-the-modulation-surface.md`                             | Heard equals value plus modulation, across internal modulators, host modulation, per-note modulation and CC learn  | Parameters and the modulation surface; the Logic and REAPER finding      | What each host delivers: nothing through clap-wrapper in Logic, destructive modulators in REAPER, full delivery in Bitwig. CC learn pushes `PARAM_VALUE` flagged `DONT_RECORD`. No modulation amount is written into state. Bitwig is acquired in Phase 3.                                                                                              |
| 0018 | `0018-tuning-tables-and-a-reed-family.md`                    | Tuning is internal tables plus per-note tuning; continuous pitch moves along a reed family                         | Tuning and continuous pitch                                              | The `f ∝ t / L²` family, and why a bare frequency sweep leaves the region Puranik found stable. The two-key mapping drives the same family. Draft `clap.tuning` has no host, and MTS-ESP is deferred because it adds C++ to the build.                                                                                                                  |
| 0019 | `0019-a-pressure-blown-indian-harmonium-and-nothing-else.md` | Savera models a pressure-blown Indian hand harmonium and nothing else                                              | What we are building; Explicitly deferred                                | The refusals, each with its reason: sample playback, waveguides, the European harmonium, and the American reed organ.                                                                                                                                                                                                                                   |
| 0020 | `0020-defer-the-gui.md`                                      | The GUI is deferred; the parameter set is the interface until it lands                                             | springer 0013; Risks                                                     | About fifty parameters at first release and over a hundred later, which is the strongest argument for pulling the GUI forward. `module` paths and AUv2 ordering are the mitigation. Native AppKit or Metal is decided when it lands.                                                                                                                    |
| 0021 | `0021-distribute-as-a-notarized-pkg.md`                      | One signed, notarized, stapled `.pkg` carrying both bundles, built locally and never in CI                         | fosforo 0014                                                             | The version stays `0.0.0` until v0.1.0. `scripts/build-installer` refuses to package unless the four version sites agree.                                                                                                                                                                                                                               |
| 0022 | `0022-stamp-provenance-without-namespacing-identity.md`      | Every build stamps its branch, commit and dirty state; nothing about identity varies                               | fosforo 0018                                                             | Savera is the third plugin installing into the same `~/Library/Audio/Plug-Ins` tree, so the multi-worktree hazard is live from the first install. Every `install-*` step prints the hash and provenance of what landed.                                                                                                                                 |
| 0023 | `0023-one-note-port-and-one-stereo-output.md`                | One note input port declaring CLAP, MIDI and MIDI-MPE with CLAP preferred, and one stereo output port              | Note input; the rescan finding                                           | Dialects, preference and channel count are rescan-gated and fixed before the first release. Voices match on the full tuple. clap-wrapper's fixed two-semitone bend in Logic is accepted, and glides there go through the two-key mapping and the tuning tables.                                                                                         |

### 0.10 Notes and the deviation record

Four files, all living documents that are corrected in place.

**`docs/notes/README.md`** adapts fosforo's. It states the three-way distinction the build plan asks for: a note records how some part of the project currently behaves and is corrected in place; an ADR records a decision and is superseded by amendment; a plan in `done/` is a historical record and is never corrected. It explains that `AGENTS.md` is loaded into every session in full and a note is not, and it carries the index table of notes, each with "read it before".

**`docs/notes/linters.md`**:

- Prettier formats every file type it supports and markdownlint verifies Markdown. Run `npm ci` first, then `npm run format`, then `npm run lint:md`. Never pass `--fix` to `markdownlint-cli2`, and why. `lint-and-fix` and `write-markdown`'s fix-mode step are satisfied by `npm run format`.
- The Homebrew copies are different versions from the pins, with the measured figures.
- `docs/design/` is excluded from both tools, the two lists are kept in step by hand, the negative controls with their measured results, and how to re-run them.
- Why `package-lock.json` is in `.prettierignore` although Prettier does not flag it.
- `MD049` off, `MD060` aligned, and `embeddedLanguageFormatting` off, each with its reason.
- What `markdownlint-rule-relative-links` checks. The pull request template links `CONTRIBUTING.md` by absolute URL, because GitHub resolves a template's relative links against the pull request page rather than the file.
- typos: words are added only when flagged; `extend-words` versus `extend-identifiers` (fosforo's pluralised-acronym trap); the `spellchecker:off` span; and that a word typos flags inside `docs/design/` is suppressed in `typos.toml`, never fixed in the brainstorm.

**`docs/notes/ci-workflows.md`**:

- The three workflows and what each job checks.
- Why none carries `paths-ignore`, why `pull_request:` is bare, and why every caller job is named for its tool.
- The gh-actions release each reusable call is pinned to, and what that release fixed that matters here (#83, #85, #87).
- The shell job's vacuous pass in Phase 0, and gh-actions #86 as the Phase 1 hazard.
- The timeouts: carried in 0.4, then re-measured from this repository's first runs and recorded with their run ids.
- How the typos pin moves, since Dependabot cannot see it.
- fosforo's rule against job or run counts in prose about CI.

**`docs/notes/skill-deviations.md`** records, for each catalog issue, every rule this project wrote by hand that the skill would otherwise supply, so the skills can be written from this project. The inclusion rule is stated at its top: an issue gets an entry when Phase 0 wrote any of its rules into a file by hand, whether as configuration or as a recorded decision. Each entry says what was written, where, and where it diverges from the issue body. Later phases append to the same entries.

| Issue | Skill                        | What Phase 0 wrote by hand                                                                                                                                                                                                                                                                             |
| ----- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| #339  | `scaffold-zig-cli`           | The Zig permissions in `.claude/settings.json`, the Zig ignore entries beyond `scaffold-new-repo`'s template (`zig-pkg/`), and the `.editorconfig` Zig section. It diverges by also allowlisting read-only lint and GitHub commands. Related: #364 and #352.                                           |
| #340  | `set-up-macos-signing`       | ADR 0021's decision and the `*.p8` ignore rule. Implementation arrives in Phases 1 and 7.                                                                                                                                                                                                              |
| #341  | `stamp-build-provenance`     | ADR 0022, adopted from fosforo 0018. Implementation arrives in Phase 1.                                                                                                                                                                                                                                |
| #342  | `set-up-clap-validation`     | ADR 0003's requirement that both bundles pass `clap-validator`. gh-actions v3.2.0's `set-up-clap-validator` now supplies the installation half, so Phase 1 consumes it.                                                                                                                                |
| #343  | `write-realtime-audio-code`  | ADR 0007's rules, and `zig.instructions.md`'s review rules built on them. It diverges from the issue in two places: savera spawns no thread, so its verification is a source canary rather than Thread Sanitizer, and trust boundaries use a runtime `if` because the shipped build removes `assert`.  |
| #344  | `scaffold-clap-audio-plugin` | ADRs 0003, 0004, 0005 and 0023, and the identifiers table in `AGENTS.md`. It diverges twice: passing `AUV2_INSTRUMENT_TYPE` is not "belt and braces" but makes the `features[0]` mapping unreachable, and a comptime import assertion cannot enforce a seam, so ADR 0005 names a run-time source walk. |
| #345  | `plant-defects`              | Nothing re-derived. The skill shipped after the build plan was written, and Phase 0's plant table follows its four columns directly.                                                                                                                                                                   |
| #346  | `check-ci-workflows`         | No `paths-ignore` on any Phase 0 workflow, a bare `pull_request:`, `workflow_dispatch:`, caller jobs named for their tool, the shell job's vacuous pass recorded rather than counted, and carried timeouts re-measured. The actionlint and shellcheck item is resolved upstream by gh-actions v3.2.0.  |
| #347  | `write-phased-build-plan`    | The `AGENTS.md` rule that the master plan never moves, because `commit` and `pr` do not implement the exemption; this phase plan's shape; the roadmap table mirrored into the README with a Status column; and issues filed when a phase is planned.                                                   |

A second section records deviations from installed skills that have no catalog issue of their own: `bootstrap-project` not used, `scaffold-new-repo` scoped down, `set-up-linters`' typos gap (#353) and its lint template's `paths-ignore`, `set-up-secret-scanning`'s `scan` job name, and `lint-and-fix` replaced by `npm run format`.

Before committing, search `cboone/agent-harness-plugins` for each defect in that second section and file, with `create-issue`, only the ones no open issue already covers. Today that means the `bootstrap-project` fix that shipped without a version bump; `set-up-linters`' `paths-ignore` template and `set-up-secret-scanning`'s job name fall under #346's scope unless its body turns out not to cover them. Record every issue number in the note.

### 0.11 AGENTS.md

Replace the stub with a hub in fosforo's post-split shape, written for savera:

- **Overview**: what Savera is, in the build plan's two goals (realism as the foundation, modulation as the point), and the product naming with its pronunciation.
- **Current state**: Phase 0 of eleven, what exists, and that nothing builds by design. The master plan is at `docs/plans/2026-09-13-savera-build-plan.md` and never moves.
- **Non-negotiables**: fosforo's instruction not to relitigate these in review but to supersede them with a new ADR, then one line per ADR, 0001 to 0023, each linking its file.
- **Identifiers, which are permanent**: the build plan's table, since changing any of them after release breaks every saved project.
- **Structure**: what exists today as a short tree, with a pointer to the build plan's [Source layout](../2026-09-13-savera-build-plan.md#source-layout) for what is planned, rather than a copy that would drift.
- **Development**: `npm ci`, `npm run format`, `npm run format:check`, `npm run lint:md`, `typos`, `actionlint` with shellcheck on `PATH`, and `gitleaks detect --no-banner`.
- **Rules**: only traps whose omission causes silent damage or a wrong pass. The master plan never leaves `docs/plans/`. `docs/design/` is preserved verbatim, and nothing in CI would catch a hand edit. Run `npm ci` before any text check, and never pass `--fix` to markdownlint-cli2. `docs/plans/done/` is a historical record and `docs/notes/` is living. Nothing under `verification/` is ever committed. A corrected figure is grepped across the repository.
- **Where the depth lives**: the pointer table into `docs/notes/`.
- **Maintaining this file**: fosforo's four questions for deciding where something belongs, and the budget. `AGENTS.md` stays under 30,000 characters, fosforo's refusal threshold, derived from Claude Code's 40,000-character floor for a memory file. That is checked by hand at each phase pull request, since the automated check was declined.

### 0.12 Review instructions

- **`.github/copilot-instructions.md`**: a pointer to `AGENTS.md` and `docs/notes/`, then review bullets. Anything under `docs/` has its own instructions, as do workflows and Zig. `printWidth: 10000` is intentional. Omitted relative pronouns are intentional, in fosforo's wording. The master plan at the `docs/plans/` root is permanent and is not a misfiled plan.
- **`.github/docs.instructions.md`**, `applyTo: "docs/**/*.md"`, adapted from fosforo: notes are living and their staleness is a defect; done plans are historical records, including their `path:line` citations and snippets; plans and ADRs state measured facts at a point in time; anchored and present-tense figures may differ; ADRs are superseded, not edited; refusals are recorded on purpose; prose is one line per paragraph; Prettier owns table shape. It adds a savera rule: **the brainstorm is preserved verbatim and superseded in places by the build plan's findings.** Do not flag its 8x oversampling figure, its subcritical jump, its Q of 95, its 4 to 6 reed polyphony, its in-rank coupling, its arXiv citation, its bare URLs or its formatting, and do not ask for it to be reconciled with the build plan.
- **`.github/actions.instructions.md`**, `applyTo: ".github/workflows/*.yml,.github/dependabot.yml"`: SHA pins with version comments; version and checksum pins together for typos; `install-pinned-tool`'s exact `archive-member` is stricter, not riskier; no `paths-ignore` in `text-lint.yml`, `gitleaks.yml` or `trufflehog.yml`; `package.json` exists for the text tools only; no `ci.yml` until Phase 1; Ubuntu unless macOS is needed.
- **`.github/zig.instructions.md`**, `applyTo: "**/*.zig,**/*.zig.zon"`, short, since no Zig exists yet: nothing under `src/model/` names the CLAP bindings, enforced by a run-time source walk; fixed capacities on the audio path are not premature optimization; parameter ids are permanent, gaps included; layout assertions are not redundant; trust boundaries use a runtime `if`, never `assert`; the solver's `.strict` float mode and `f64` state are deliberate; no line-length limit.

`cmake.instructions.md` and `shell.instructions.md` wait for Phase 1, when their `applyTo` globs have files to match.

Then invoke `clean-up-agent-config` in audit mode over the hub-and-spoke layout (`AGENTS.md`, the `CLAUDE.md` symlink, `.claude/settings.json`, and the flat `.github/*.instructions.md` layout the siblings use). Apply nothing that contradicts this plan, and record anything it finds in the outcomes.

### 0.13 Community files

- **`CONTRIBUTING.md`**, adapted from springer: a single-developer project; read `AGENTS.md` first; issues through the tracker, security through `SECURITY.md`; twenty-three settled decisions with refusals recorded in ADR 0019; development setup for Phase 0 (Node 22 or later, `npm ci`, typos, actionlint with shellcheck, gitleaks, and why the Homebrew copies of the Node tools must not be used); the brainstorm must not be reformatted or edited; code style (one line per paragraph, Prettier owns layout, never `--fix`); **commit subjects in Conventional Commits, pull request titles in sentence case with no type prefix**; GPG signing; branch naming; the pull request process.
- **`CODE_OF_CONDUCT.md`**: Contributor Covenant 3.0, verbatim from fosforo, including its contact address.
- **`.github/SECURITY.md`**: private vulnerability reporting, answered on a best-effort basis by a single maintainer, with no hour commitments. The threat model is that of an in-process audio plugin: hostile or malformed saved state, preset files and preset discovery locations, raw MIDI and controller input including RPN sequences, out-of-range or non-finite parameter values, and host sample rates or block sizes outside what `activate` accepts. Out of scope: CLAP, clap-wrapper and hosts themselves, and crashes that need a host violating CLAP's threading contract.
- **`.github/PULL_REQUEST_TEMPLATE.md`**: `<!-- markdownlint-disable MD041 -->`, with a checklist seeded with the Phase 0 gate that grows a line per phase. The rows: CONTRIBUTING read (absolute URL); `npm ci`, then `npm run format:check` and `npm run lint:md` clean, never `--fix`; `typos` clean; `actionlint` silent with shellcheck on `PATH` if a workflow changed; `shfmt -d` and `shellcheck` silent if a script changed; `git diff --exit-code origin/main -- docs/design/` empty; `AGENTS.md` under 30,000 characters with `CLAUDE.md` still its symlink; the master plan still at the `docs/plans/` root; for a phase pull request, the phase plan moved to `done/` and the master plan's outcomes subsection added; `CHANGELOG.md` updated for a user-facing change; a superseding ADR for a changed decision.

### 0.14 README and the repository description

Rewrite `README.md` on fosforo's shape:

- **Title and tagline**, using the repository description, with a line for the name: Hindi and Urdu for "dawn", said sa-VEH-ra.
- **Status**: early development, nothing builds or loads, no release.
- **Roadmap**: the build plan's eleven phases as a table of Phase, Scope and Status, with one-line scopes, linking the build plan for exit criteria and saying issues are filed one phase at a time. Phase 0 reads "In progress" until 0.17 sets it to "Complete".
- **Requirements, as planned**: macOS 11 or later on Apple Silicon, with Logic Pro loading the Audio Unit and REAPER or Bitwig loading the CLAP.
- **Installation and usage**: none yet; v0.1.0 ships a signed package in Phase 7.
- **Contributing** and **License**.

Set the description with `gh repo edit cboone/savera --description "A modulatable synthesizer built on a physical model of the Indian hand harmonium (peti), for macOS (CLAP and AUv2)"`, then confirm it with `gh repo view cboone/savera --json description`.

### 0.15 The local gate and the local plants

Run the local gate from [Verification](#verification) on a clean tree first. Then work the [plant table](#plant-table) with `plant-defects`' discipline. Commit every check before planting against it. Plant one defect at a time in the working tree, and restore it with `git restore` before the next; never stash in this worktree. Record what actually happened, not what was expected. Confirm that the finding named in each row is the one that fired, not a neighbouring rule.

Invoke `pin-everything` as an audit only: confirm that no `uses:` reference floats, that every npm pin is exact, and that the typos pin carries its checksum. Decline its optional drift-audit script. Record any gap in the outcomes rather than widening Phase 0.

### 0.16 Review, push, pull request, and the CI plant

1. Invoke `review-branch` to check the branch against this plan before anything leaves the machine.
1. Push `feature/phase-0`, which has no upstream yet, with `git push -u origin feature/phase-0`.
1. Open the pull request with the `pr` skill, titled `Establish the repository foundation for Phase 0` in sentence case, as `CONTRIBUTING.md` now requires. The tree is committed, so the skill has no plan file to move; confirm afterwards that it moved nothing under `docs/plans/`.
1. Wait for every check to go green on the clean head, then read the positive controls from the logs rather than from the green status: `actions / actionlint` reports actionlint 1.7.12 and ShellCheck 0.11.0; `typos` reports 1.50.1; `text / Text lint` shows `npm ci` in the workspace and markdownlint linting a non-zero file count; `shell / Shell lint` shows discovery finding nothing and every later step skipped. That last one is the vacuous pass, recorded as such.
1. **The CI plant.** The `relative-links` rule resolves through the workspace's `node_modules` only in CI's consumer-versions path, which no local run exercises, so the CI instruments are planted once for real. Commit `test: plant one defect for each CI lint instrument` containing plants P1, P2, P3, P4 and P8 at once (each targets a different job, or a different rule within `text`). Push it, wait for the run to finish (the concurrency group cancels an in-progress run on the same ref), and confirm that each job is red for its own plant. Then `git revert -S --no-edit HEAD`, push, and confirm green again. Both commits stay in history as the record. Never plant a secret-shaped string in a pushed commit: this repository is public.
1. Invoke `monitor-pr` until checks are green, Copilot has reviewed the current head with nothing left open, and the pull request is mergeable, using `resolve-copilot-pr-feedback` for its comments.

### 0.17 Outcomes, and the move to done

1. Append `### Phase 0 outcomes (complete)` to the master plan at the end of `## Phases`, in springer's shape: the date and a link to this plan in `done/`; the step table with a status column; everything that landed differently from this plan; the exit criteria with measured results and CI run ids; the gaps stated rather than counted (the shell job's vacuous pass, the brainstorm digest and `AGENTS.md` size checked by hand with their values, nothing builds); and the facts that moved since 2026-09-13 (gh-actions v3.2.0, #345 closed).
1. Update the Status line of the Risks entry "gh-actions release timing" in place: resolved on 2026-09-14, because v3.2.0 ships `set-up-clap-validator`. Change nothing else in the master plan outside the new subsection.
1. Fill in this plan's plant table and add an Outcomes section at its foot.
1. Set Phase 0 to "Complete" in the README roadmap, and update `AGENTS.md`'s Current state to match.
1. Run `npm run format`, then the local gate again, since the master plan's tables realign. Commit `docs: record Phase 0 outcomes`.
1. `git mv docs/plans/todo/2026-09-14-savera-phase-0-repository-foundation.md docs/plans/done/`, and commit `docs: move the Phase 0 plan to done`. Its relative links keep resolving from `done/`, which sits at the same depth.
1. Push, confirm green, and run `git diff --name-status origin/main...HEAD -- docs/plans/`. The expected output is exactly one added file in `done/` and one modified master plan, with no rename of the master plan.

The owner merges. Merge with a merge commit, not a squash, so that the CI plant and its revert survive as the record.

## Files

```text
.claude/settings.json                     permissions from Decisions
.editorconfig                             adapted from fosforo; shell section waits for Phase 1
.github/
  PULL_REQUEST_TEMPLATE.md                checklist seeded with the Phase 0 gate
  SECURITY.md                             best effort, audio-plugin threat model
  actions.instructions.md
  copilot-instructions.md
  dependabot.yml                          github-actions and npm
  docs.instructions.md                    gains the brainstorm rule
  zig.instructions.md                     seeded, short
  workflows/
    gitleaks.yml                          no paths-ignore
    text-lint.yml                         text, shell, actions, typos; no paths-ignore
    trufflehog.yml                        no paths-ignore
.gitignore                                merged, then extended
.gitleaks.toml                            Zig, build directories, fixture entropy
.markdownlint-cli2.jsonc                  fosforo's rules plus relative-links
.npmrc                                    engine-strict
.prettierignore                           docs/design/, package-lock.json, build outputs
.prettierrc.json                          embeddedLanguageFormatting off
AGENTS.md                                 the hub
CHANGELOG.md                              an empty Unreleased section
CLAUDE.md -> AGENTS.md
CODE_OF_CONDUCT.md                        Contributor Covenant 3.0
CONTRIBUTING.md                           sentence-case pull request titles
LICENSE                                   unchanged
README.md                                 rewritten
docs/
  adr/
    README.md                             the index
    0001-*.md to 0023-*.md
  design/
    peti-physical-model-brainstorm.md     unchanged, byte for byte
  notes/
    README.md
    ci-workflows.md
    linters.md
    skill-deviations.md
  plans/
    2026-09-13-savera-build-plan.md       permanent; gains Phase 0 outcomes
    done/
      2026-09-14-rename-project-to-savera.md
      2026-09-14-savera-phase-0-repository-foundation.md   this plan, after 0.17
    todo/.gitkeep
package-lock.json                         the pin
package.json                              markdownlint-cli2, the relative-links rule, Prettier
typos.toml                                no words until one is flagged
verification/.gitkeep
```

## Plant table

The third and fourth columns are filled in during 0.15 and 0.16, from what happened. P5 and P6 were measured while planning.

| #   | Planted defect                                                                                                           | Instrument expected to catch it                                      | What actually happened                   | Test that covers it now                                                       |
| --- | ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- | ---------------------------------------- | ----------------------------------------------------------------------------- |
| P1  | A misspelled word in an ADR body, locally and in the CI plant                                                            | `typos`, and the `typos` job                                         |                                          |                                                                               |
| P2  | One table row's pipes misaligned in a note, locally and in the CI plant                                                  | Prettier `--check`, and markdownlint `MD060`, each on its own        |                                          |                                                                               |
| P3  | A relative link to a file that does not exist, in `AGENTS.md`, locally and in the CI plant                               | `relative-links`                                                     |                                          |                                                                               |
| P4  | A link to an existing ADR with a fragment that names no heading, locally and in the CI plant                             | `relative-links`, fragment check                                     |                                          |                                                                               |
| P5  | Prettier over the brainstorm with `--ignore-path /dev/null`, then with the ignore file                                   | Prettier; the second run must skip the file                          | Measured while planning: flagged, exit 1 |                                                                               |
| P6  | markdownlint over the brainstorm with `docs/design/**` removed from `ignores`                                            | markdownlint                                                         | Measured while planning: 13 issues       |                                                                               |
| P7  | One byte of the brainstorm changed                                                                                       | Nothing in CI, by decision                                           |                                          | By hand: the recorded SHA-256 and `git diff --exit-code`                      |
| P8  | An unquoted variable (`SC2086`) in a `run:` step of `text-lint.yml`, locally and in the CI plant                         | actionlint with shellcheck                                           |                                          |                                                                               |
| P9  | P8 again, locally, with shellcheck removed from `PATH`                                                                   | Nothing; the expected result is exit 0, the vacuous pass             |                                          | The pinned shellcheck install and version report in `lint-github-actions.yml` |
| P10 | A randomly generated, syntactically valid access key id in an uncommitted Markdown file, with `gitleaks detect --no-git` | gitleaks                                                             |                                          |                                                                               |
| P11 | The same string in an uncommitted `fixtures/plant.wav`                                                                   | Nothing; the fixture allowlist entry, expected silent                |                                          |                                                                               |
| P12 | An extra space after a colon in a workflow's YAML                                                                        | Prettier over YAML                                                   |                                          |                                                                               |
| P13 | P3 and P4 again, with `customRules` removed from `.markdownlint-cli2.jsonc`                                              | Nothing; expected green, proving no built-in rule was the one firing |                                          |                                                                               |
| P14 | `CLAUDE.md` replaced by a copy of `AGENTS.md`                                                                            | Nothing, by decision                                                 |                                          | By hand: `test -L CLAUDE.md`                                                  |
| P15 | A shell script under `scripts/` with `SC2086`                                                                            | The `shell` job                                                      | Not planted: Phase 0 adds no script      | Phase 1's gate plants it                                                      |

P11 is the row that tests the allowlist rather than trusting its regex: gitleaks may report working-tree paths with or without a leading `./`, and `^fixtures/` matches only one of them. If P11 is flagged, the regex is corrected and both P10 and P11 are re-run.

P13 is `plant-defects`' step of planting against the judge. `MD051` checks link fragments within a single file, so a cross-file fragment should be seen by the custom rule alone; the row confirms that rather than assuming it.

## Verification

The local gate, run in 0.15 and again in 0.17:

```bash
npm ci
npm run format:check                 # clean
npm run lint:md                      # 0 issues; record the file count
typos                                # 0 findings
shellcheck --version && actionlint   # shellcheck first, or actionlint skips run: blocks
gitleaks detect --no-banner          # no leaks
test -L CLAUDE.md && [ "$(readlink CLAUDE.md)" = AGENTS.md ]
wc -c AGENTS.md                      # under 30000
shasum -a 256 docs/design/peti-physical-model-brainstorm.md
                                     # ee012eff0fa93b1ae73dc311e516e1c709ab30b15319bfbfcd0f1bb91ce13529
git diff --exit-code origin/main -- docs/design/
npx prettier --check --ignore-path /dev/null docs/design/peti-physical-model-brainstorm.md
                                     # must fail: the negative control
git diff --name-status origin/main...HEAD -- docs/plans/
```

In CI, on the pull request: `text / Text lint`, `shell / Shell lint`, `actions / actionlint`, `typos`, and the gitleaks and TruffleHog rows all green on the final head, with the positive controls read from the logs as 0.16 describes, and the CI plant's run red in `text`, `typos` and `actions` for their own plants.

By hand, and recorded in the outcomes:

- Every catalog issue from #339 to #347 has an entry in `docs/notes/skill-deviations.md`, and every issue filed in 0.10 is linked from it.
- The repository description reads as Decisions states.
- The master plan is still at the `docs/plans/` root, carries the new outcomes subsection, and its only other change is the one Risks status line.

Two things are stated in the outcomes rather than counted as coverage:

- **The shell job passes by finding nothing.** It is wired for Phase 1 and exercised there.
- **Nothing builds.** There is no `zig build`, no bundle and no `clap-validator` run, by design.

## Out of scope

Deferred to Phase 1, so these read as sequencing rather than omission: `build.zig`, `build.zig.zon`, `src/`, `cmake/`, `macos/Info.plist`, `scripts/`, `ci.yml` and the `set-up-ci` invocation that writes it, shebang-based shell discovery, the `.editorconfig` shell section, `cmake.instructions.md` and `shell.instructions.md`.

Deferred to Phase 2: `ruff.toml`, the `python` job, the Python ignore entries, `fixtures/`, and the REUSE decision.

Deferred to the planning of each phase: the milestones and issues for Phase 1 and Phase 2.

Not done at all: `bootstrap-project` as the orchestrator; cspell and yamllint; `optimize-runner-usage`, which would add the `paths-ignore` these workflows deliberately refuse; `lint-and-fix`; a pre-seeded typos vocabulary; branch protection; `.git-blame-ignore-revs`.

## Follow-ups

- After merge, confirm Dependabot parsed `.github/dependabot.yml` from `main` and opened no configuration error.
- After merge, confirm `text-lint.yml`, `gitleaks.yml` and `trufflehog.yml` ran on the `push` to `main`.
- Once the job names have held through Phase 1, decide whether `main` gets branch protection with these checks required.
