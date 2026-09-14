# Rename the project to savera

## Context

The project is being renamed from cellino to savera, completely: the repository, the product name, the permanent plugin identifiers, file names, local checkout paths, the terminal session, and Claude Code session history. Svara was the first choice, but Orchestral Tools already sells an Indian-instrument library under that name, so Stage 0 chose Savera instead: Hindi and Urdu for "dawn", said sa-VEH-ra. The same change moves the reverse-DNS vendor prefix from `com.catamount` to `com.catamountaudio`.

Now is the easiest point to do it. `main` holds three tracked files and no code. There are no GitHub issues, pull requests, releases, topics, or description. Nothing has shipped, so no host project has saved any of the identifiers the build plan calls permanent. Phase 0 is next, and it creates twenty-three ADRs, `AGENTS.md`, and CI, followed by `build.zig`; each of those would carry the old name. The `feature/phase-0` worktree already exists but has no commits, so renaming first means Phase 0 starts under the new name.

Outcome: the old name survives only in git history and in this plan. `cboone/savera` is the repository, and GitHub redirects the old URL to it. The checkout, the worktree, the tmux session, and Claude Code history all live under `savera` paths.

## Preflight findings

Measured on 2026-09-14.

- `cboone/cellino` is public, not archived, and not a fork: there is no `upstream` remote, and `origin` is `cboone/cellino`. `cboone/savera` does not exist.
- Tracked occurrences: `README.md` 1 line, `docs/design/peti-physical-model-brainstorm.md` 1 line, `docs/plans/2026-09-13-cellino-build-plan.md` 30 lines. The build plan also has the derived codes `Clno` (3 lines) and `CLNO` (2 lines), and the vendor prefix `com.catamount` on 2 lines (the CLAP id and the preset provider id). No other casing of the name appears.
- Nothing outside the repo refers to it: no match in `~/Development/fosforo`, `~/Development/springer`, other projects under `~/Development`, `~/.config`, or GitHub code, issues, and pull requests under `cboone`.
- The siblings use the old vendor prefix: fosforo declares `com.catamount.fosforo` in `src/clap/plugin.zig`, and springer records `com.catamount.springer` in `AGENTS.md`. Neither has a tag or a release. See [Follow-ups](#follow-ups).
- Git history needs no rewrite, because no commit subject or branch name contains the name.
- Local state tied to the path:
  - The workmux worktree `~/Development/cellino__worktrees/phase-0` on `feature/phase-0`, which is clean and zero commits ahead of `main`.
  - The tmux session `cellino`, with windows `cellino` and `𝚲 phase-0`, each running Claude Code.
  - Three Claude Code project directories, `~/.claude/projects/-Users-ctm-Development-cellino`, `...-cellino--worktrees-phase-0`, and `...-cellino--worktrees-plan-phases`. They hold transcripts; their memory directories are empty.
- Lint baselines:
  - The build plan passes `prettier --check`. `README.md` fails it because it has no trailing newline.
  - `markdownlint-cli2`, with no repo config, applies its 80-column default and reports 290 MD013 findings on the build plan and nothing else, plus MD047 on `README.md`.
  - `typos` is clean, and accepts `Savera.clap`, `savera_impl`, `SAVERA_FORMATS`, `SAVERA_SIGNING_IDENTITY`, `Svra`, `SVRA`, and `com.catamountaudio.savera.presets`.
- Name clash: Orchestral Tools sells [Svara](https://www.orchestraltools.com/svara), an Indian film orchestra sample library for SINE Player, shipped as AU, VST, and AAX. That is why Stage 0 exists.

## Stage 0: choose the name

### Decision

Savera, chosen on 2026-09-14 after three rounds. It keeps the soft, breathy sound that drew the project to Svara. Nothing in audio or music uses it, and no live US trademark covers software or instruments.

Neither the USPTO nor India's trademark registry accepted automated searches, so neither has been searched directly. That search is a follow-up for before v0.1.0 rather than a gate on Stage 1: nothing becomes permanent until that release, and the US records mirrored by TrademarkElite show nothing live in software or instruments.

The rest of this stage records how the name was chosen.

### Criteria

1. No existing audio product uses the name, especially Indian-instrument and harmonium products: NUSofting's Peti SA, Swar Systems' SwarPlug, Orchestral Tools' Svara, and the Kontakt harmonium libraries.
1. It relates to what the plugin is: the peti's air, reeds, or box, or the continuous pitch that only the model can produce.
1. It has one obvious ASCII spelling, because the name becomes the repository, the bundle, the CLAP id, and a four-letter AU subtype. A diacritic may appear in the display name only, as in Fósforo.
1. An English speaker can say it, and can spell it after hearing it.
1. It is not a deity, a sacred term, a person, or an instrument maker's brand, and it has no stronger unrelated meaning.
1. Its AU subtype, derived the way the siblings' were, differs from `Fsfr` and `Sprg`.

### Round 1 candidates

The web check column comes from general web searches run on 2026-09-14. Not finding a product that way is weak evidence; the finalist checks go further.

| Name      | Meaning                                                         | For                                                                           | Against                                                                    | Web check                                |
| --------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| Meend     | Hindi, a glide between notes in Hindustani music                | A real peti cannot glide and this model can, so it names the synthesizer goal | Names a feature rather than the instrument; also spelled "mind"            | No product found                         |
| Petika    | Sanskrit and Hindi, a small box; the root of peti               | Names the instrument itself, and is a diminutive as cellino was               | Close enough to NUSofting's Peti to look derivative in the same niche      | No product found                         |
| Andolan   | Hindi, a slow oscillation around a held note                    | A reed is an oscillator, and this ornament is modulation of a held note       | Its everyday Hindi meaning is a political movement                         | No product found                         |
| Dhaunkni  | Hindi and Urdu, bellows                                         | The bellows set loudness and feed the shared air supply                       | Two common spellings (dhaunkni, dhonkni); hard to spell from hearing       | No product found                         |
| Jali      | Hindi, the lattice cover on the harmonium's front               | The jali open and closed are the enclosure filter's two presets; short        | Also the Mandinka word for a griot, from an unrelated musical tradition    | An Android app is named "Harmonium Jali" |
| Kharaj    | Hindi, the lowest vocal register                                | Fits the bass bank and deep bellows pressure                                  | The "kh" is hard for English speakers; also a Bengali actor-singer's name  | No product found                         |
| Hawa      | Hindi and Urdu, air                                             | Short and easy to say                                                         | A very common word in several languages, so it is hard to make distinctive | No product found                         |
| Vaaja     | Vernacular name for the harmonium in several Indian languages   | Names the instrument                                                          | Generic, and also spelled vaja and baja                                    | No product found                         |
| Linguetta | Italian for "little tongue", the vibrating blade of a free reed | Matches the family's Romance-language names and cellino's diminutive          | No connection to India                                                     | No product found                         |

Ruled out before round 1:

- Svara: Orchestral Tools' library.
- Peti: NUSofting's Peti SA.
- Swar: Swar Systems' SwarPlug.
- Shruti and Riyaz: names of established Indian music practice apps.
- Samvadini: another name for the harmonium itself.
- Naad, Prana, and Vayu: spiritual weight and heavy brand use.
- Surpeti: a different drone instrument.
- Dwarkin: a Kolkata harmonium maker.

### Finalist checks

Each round's shortlist was checked against:

- KVR, Plugin Boutique, and the App Store, with site-restricted searches.
- General brand searches and web searches for trademarks, plus US trademark records through TrademarkElite.
- `gh`, for `cboone/<name>` and any other repositories with the same name.
- `typos`.
- Pronunciation and spelling from hearing, and the derived AU subtype, state magic, CLAP id, and bundle id.

### Round 1 finalists

Meend and Petika were shortlisted, then set aside after these checks.

| Check                      | Meend                                                                                                                                                                                                         | Petika                                                                                                                                                                                         |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Plugins and apps           | None named Meend. Indian-music apps list it as a feature: an Indian classical practice app's meend glides, SwarMeter's meend graph, and moForte's physically modelled Naada Shehnai.                          | None named Petika.                                                                                                                                                                             |
| Other uses of the name     | Two small GitHub instruments: `p3r7/meend` (Lua, 2024, empty README) and `AnupKatalkar/meend` (a browser webcam gesture instrument, August 2026, named for the same reason: "every control here is a glide"). | Unrelated small brands outside music: a plant-based leather bag (Vyakti India), Pizza Petika, a South African furniture maker, an Indian paper LLP. Also the name of a class of Tantric texts. |
| Trademark (web search)     | No MEEND record found; the nearest are MEND and MEJEND, both unrelated.                                                                                                                                       | No PETIKA record found.                                                                                                                                                                        |
| Main risk                  | In this niche the word describes a feature, so it makes a weak trademark and search results bring up every app that advertises meend glides.                                                                  | In the same niche as NUSofting's Peti SA, a name built on "peti" could look derivative or be confused with it.                                                                                 |
| Say and spell              | One pronunciation, rhymes with "fiend"; spelled as heard.                                                                                                                                                     | Stress is ambiguous to English speakers (PET-i-ka or pe-TEE-ka).                                                                                                                               |
| GitHub `cboone/<name>`     | Free.                                                                                                                                                                                                         | Free.                                                                                                                                                                                          |
| `typos`                    | Clean.                                                                                                                                                                                                        | Clean.                                                                                                                                                                                         |
| AU subtype and state magic | `Mend` and `MEND`. Like `Clno`, it keeps the name's letters in order.                                                                                                                                         | `Ptka` and `PTKA`.                                                                                                                                                                             |
| CLAP id and bundle id      | `com.catamountaudio.meend`, `com.cboone.meend`                                                                                                                                                                | `com.catamountaudio.petika`, `com.cboone.petika`                                                                                                                                               |

### Round 2 candidates

This round covers what harmonium players do, the repertoire they play, the instrument's European ancestors, and Romance-language words for breath and bellows to match fosforo. The web checks are general searches from 2026-09-14.

| Name          | Meaning                                                                                           | For                                                                 | Against                                                                 | Web check        |
| ------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------- | ---------------- |
| Surila        | Hindi, "tuneful" or "melodious"                                                                   | An everyday word for sounding good; pleasant to say (soo-REE-la)    | Also spelled sureela and surilaa                                        | No product found |
| Pilu          | The light raga at the heart of thumri, a repertoire the harmonium accompanies                     | Short, easy, and secular                                            | Names one raga, for an instrument built for any tuning                  | No product found |
| Kajri         | A monsoon song form from Uttar Pradesh and Bihar                                                  | Evocative and secular                                               | Many albums are titled Kajri, so search results are crowded             | No product found |
| Murki         | A quick cluster of grace notes in Hindustani music                                                | Harmonium players are known for fast murkis; one spelling           | Sounds like the English "murky", a poor word for a sound                | No product found |
| Pakad         | "Grip", the signature phrase that identifies a raga                                               | Distinctive                                                         | Hard-sounding in English; raga apps use it as a feature name            | No product found |
| Mandra        | The lower octave                                                                                  | Fits the bass bank                                                  | Says little about the instrument's character; close to MNDALA and MNTRA | No product found |
| Fiato         | Italian, "breath"; wind instruments are strumenti a fiato                                         | Matches the family's Romance-language names; short                  | No link to India                                                        | No product found |
| Fuelle        | Spanish, "bellows"                                                                                | Matches Fósforo's Spanish                                           | Strongly tied to tango and the bandoneon                                | No product found |
| Physharmonica | Anton Haeckl's 1818 Viennese free-reed keyboard; the small ones were held on one arm, like a peti | A real ancestor of the peti, and "phys" hints at physical modelling | Long and awkward to say; the name of a historical instrument            | No product found |

Ruled out in round 2:

- Lehra: Lehra Studio, iLehra, and RiyazStudio Lehra already use it.
- Gunjan: a common given name.
- Taan: too common, and also spelled "tan".
- Soffietto: hard to say.

No round 2 name made the shortlist.

### Round 3 candidates

Meend and Petika missed on sound and feel, so this round starts from sound. Svara is soft and breathy: it opens with the fricatives _s_ and _v_, runs through a liquid _r_, and ends on an open _a_. Meend closes on a nasal and a stop, and Petika is three stops (_p_, _t_, _k_). Fósforo and cellino share Svara's shape: fricatives or liquids, and an open vowel at the end. So this round keeps to two or three syllables, _s_, _f_, _v_, _h_, _l_, and _r_ sounds, few or no hard stops, and a vowel ending. Web checks are general searches from 2026-09-14.

| Name     | Say it      | Meaning                                              | For                                                                          | Against                                                                                   | Web check                                        |
| -------- | ----------- | ---------------------------------------------------- | ---------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Savera   | sa-VEH-ra   | Hindi and Urdu, "dawn"                               | The closest sound to Svara; dawn is when singers practise with the harmonium | Oblique; a common word in Hindi and Urdu film song titles                                 | No product found                                 |
| Soffio   | SOHF-fyo    | Italian, "breath" or "puff of air"                   | Sounds like fósforo and cellino; breath is what the bellows supply           | No link to India; also the Italian medical term for a heart murmur                        | No product found; FFOSSO is similar              |
| Soffiato | sof-FYAH-to | Italian, "blown", as in vetro soffiato (blown glass) | The build plan calls the peti a pressure-blown free-reed box                 | One hard _t_; no link to India                                                            | No product found                                 |
| Surila   | soo-REE-la  | Hindi, "tuneful" or "melodious"                      | Fits the sound profile, carried over from round 2                            | Also spelled sureela and surilaa                                                          | No product found                                 |
| Anila    | AH-nee-la   | Sanskrit, "wind" or "air"                            | Soft and flowing; air is the instrument's power source                       | A given name, and an epithet of the Vedic wind god; close to the ANINA plugin             | No product found; ANINA (CRQL) is similar        |
| Armonio  | ar-MOH-nyo  | Italian, "harmonium"                                 | Says exactly what it is, in the family's Romance-language style              | Generic, and names the European instrument ADR 0019 excludes                              | No product found                                 |
| Svasa    | SVAH-sa     | Sanskrit, "breath"                                   | Same shape as Svara, and the two words share breath as a meaning             | Also spelled shvasa and swasa; also means laboured breathing or asthma; svasā is "sister" | No product found; an Ayurvedic remedy uses Swasa |

Ruled out in round 3:

- Sospiro: Osterhouse Sounds' Sospiro Strings, a Kontakt library.
- Alito: in English, the surname of a sitting US Supreme Court justice outweighs the Italian "breath".

### Round 3 finalists

Savera and Svasa were shortlisted, and Savera was chosen.

| Check                      | Savera                                                                                                                                                                                                                                                 | Svasa                                                                                                                                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Plugins and apps           | No audio or music product. App Store hits are all outside music: Dainik Savera Times (news), two restaurant apps, a furniture maker.                                                                                                                   | No audio or music product. The App Store has Vee+ Svasa, a COVID-era health information app, and yoga apps use "swasa" in the names of breathing exercises.                            |
| Other uses of the name     | A crowded general brand in India (Savera Hotels in Chennai, Savera rice, several textile and pharmaceutical marks) and Spain's Savera Group, which makes elevator components. A common word in Hindi and Urdu film song titles.                        | Mostly wellness: yoga studios, Ayurvedic products (Swasa Sudharana), a Svasa Designs page.                                                                                             |
| US trademarks              | Nothing live in software or instruments. SAVERA, class 9 software (Savera Systems, filed 1998), abandoned in 2000. SAVERA, class 7 elevators (S.A. de Vera), cancelled in 2025. S SAVERA, a design mark for class 6 metal building materials, is live. | No SVASA record found.                                                                                                                                                                 |
| Main risk                  | So common as a word and a brand that a search for the name alone will not find the plugin.                                                                                                                                                             | One letter away from Svara, the name being avoided, in the same Indian-instrument niche. Also spelled shvasa and swasa; also means laboured breathing or asthma; svasā means "sister". |
| Say and spell              | Three open syllables, sa-VEH-ra. English readers may put the stress on the first syllable, but the spelling is fixed.                                                                                                                                  | SVAH-sa. The _sv_ opening is unfamiliar in English, and people who have heard it tend to write "swasa".                                                                                |
| GitHub `cboone/<name>`     | Free.                                                                                                                                                                                                                                                  | Free.                                                                                                                                                                                  |
| `typos`                    | Clean.                                                                                                                                                                                                                                                 | Clean.                                                                                                                                                                                 |
| AU subtype and state magic | `Svra` and `SVRA`, since Savera's letters s, v, r, a come in that order.                                                                                                                                                                               | `Svsa` and `SVSA`.                                                                                                                                                                     |
| CLAP id and bundle id      | `com.catamountaudio.savera`, `com.cboone.savera`                                                                                                                                                                                                       | `com.catamountaudio.svasa`, `com.cboone.svasa`                                                                                                                                         |

## Name mapping

| Old                                                         | New                                                       | Sites                                                                                                |
| ----------------------------------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `com.catamount.cellino`                                     | `com.catamountaudio.savera`                               | CLAP id, and the preset provider id `com.catamount.cellino.presets`                                  |
| `cellino`                                                   | `savera`                                                  | Repository, README heading, prose, `cellino_impl`, bundle id `com.cboone.cellino`                    |
| `Cellino`                                                   | `Savera`                                                  | Product and display name, `Cellino.clap`, `Cellino.component`, prose, the one word in the brainstorm |
| `CELLINO_`                                                  | `SAVERA_`                                                 | `CELLINO_FORMATS`, `CELLINO_SIGNING_IDENTITY`                                                        |
| `Clno`                                                      | `Svra`                                                    | AU subtype, in the identifiers table and both `auval` commands                                       |
| `CLNO`                                                      | `SVRA`                                                    | State magic, in the identifiers table and the state format paragraph                                 |
| `2026-09-13-cellino-build-plan.md`                          | `2026-09-13-savera-build-plan.md`                         | Master plan filename; the date stays, because it records when the plan was written                   |
| `2026-09-14-rename-project-to-svara.md`                     | `2026-09-14-rename-project-to-savera.md`                  | This plan, drafted before the name was settled                                                       |
| `cboone/cellino`                                            | `cboone/savera`                                           | GitHub repository and the `origin` URL                                                               |
| `~/Development/cellino`, `~/Development/cellino__worktrees` | `~/Development/savera`, `~/Development/savera__worktrees` | Checkout and workmux worktree root                                                                   |
| tmux session and window `cellino`                           | `savera`                                                  | Terminal                                                                                             |

`Svra` keeps Savera's letters s, v, r, a in order, the way `Clno` keeps c, l, n, o from cellino, and it stays distinct from fosforo's `Fsfr` and springer's `Sprg`. Most of the build plan's notes on these rows still hold after the change: the repository still matches the product name, the display name is plain Savera so the ASCII and display names still agree, and the state magic is still the AU subtype in uppercase.

One note stops being true. The CLAP id row says "Vendor identity shared with fosforo and springer", but the siblings still use `com.catamount`, so Stage 1 rewrites that note. The bundle identifier stays under `com.cboone`, because the build plan deliberately keeps the signing identity separate from the vendor identity. Only the two vendor identity strings move to `com.catamountaudio`.

Unchanged: AU manufacturer `Ctmn` / `Catamount`, the `com.cboone` signing prefix, AU type `aumu`, `features`, the brainstorm's filename, and git history.

## Stage 1: rename the tracked content

Work on a branch in the main checkout, not in a new workmux worktree, because Stage 3 moves the worktree root. Invoke `write-markdown` before editing. Every commit is GPG-signed and uses Conventional Commits.

1. `git switch -c chore/rename-to-savera`. Rename this plan from `2026-09-14-rename-project-to-svara.md` to `2026-09-14-rename-project-to-savera.md` (it is untracked, so a plain `mv`), then commit it: `docs: add the plan to rename the project to savera`.
1. `git mv docs/plans/2026-09-13-cellino-build-plan.md docs/plans/2026-09-13-savera-build-plan.md`, committed on its own: `docs: rename the build plan file for savera`. Committing the rename without content changes keeps `git log --follow` reliable after the content edit that comes next.
1. In the build plan, use Edit with `replace_all` (case-sensitive), in this order: `com.catamount.cellino` to `com.catamountaudio.savera`, then `CELLINO` to `SAVERA`, `Cellino` to `Savera`, `cellino` to `savera`, `CLNO` to `SVRA`, `Clno` to `Svra`. The vendor prefix goes first; after the `cellino` replacement, the string would read `com.catamount.savera` and the first pattern would no longer match it.
1. In the identifiers table, change the CLAP id row's note from "Vendor identity shared with fosforo and springer." to "Catamount Audio vendor identity."
1. Reread the changed lines where the name carries meaning:
   - the title (line 5) and Context (lines 9 and 15)
   - the ADR 0019 row
   - the identifiers table (lines 107 to 119)
   - the build architecture bullets (lines 127 to 134)
   - the state format paragraph (line 323)
   - the Phase 1 row, the verification table, and the command block (lines 374 to 478)

   Savera starts with a consonant, as Cellino does, so no article should need to change.

1. Run `prettier --write docs/plans/2026-09-13-savera-build-plan.md` to realign the tables whose column widths changed. Then run `git diff --word-diff` and confirm the diff contains only renamed tokens, the one rewritten note, and table padding. Do not run markdownlint with `--fix`, because the build plan makes Prettier the owner of Markdown formatting. Commit: `docs: rename cellino to savera in the build plan`.
1. In `README.md`, change `# cellino` to `# savera` and end the file with a newline.
1. In the brainstorm, change `Cellino` to `Savera` on line 5 with one Edit and nothing else. Do not run Prettier or markdownlint on it, so every other byte stays identical. Commit steps 7 and 8 together: `docs: rename cellino to savera in the README and brainstorm`.
1. `git mv` this plan to `docs/plans/done/` and commit: `docs: move the savera rename plan to done`. Stages 2 to 4 are operational and change no tracked file.
1. Merge the way `docs/plan-phases` was merged, as a signed local merge commit, since the repo has no pull requests: `git switch main`, `git merge --no-ff -S chore/rename-to-savera`, `git push origin main`, `git branch -d chore/rename-to-savera`.

## Stage 2: rename the GitHub repository

1. `gh repo rename savera --repo cboone/cellino --yes`.
1. `git remote set-url origin https://github.com/cboone/savera.git`, then `git fetch origin`.

GitHub redirects web, git, and API requests for `cboone/cellino` to `cboone/savera`. The redirect ends if a new repository named `cellino` is ever created under `cboone`, so do not reuse that name.

## Stage 3: move the local directories

This session runs inside `~/Development/cellino`, and the phase-0 session runs inside the worktree. Moving either directory would remove the working directory of a running session, so Stage 3 runs in a plain shell after exiting Claude Code in both tmux windows. First confirm that `git -C ~/Development/cellino__worktrees/phase-0 status` is still clean.

```bash
mv ~/Development/cellino ~/Development/savera
mv ~/Development/cellino__worktrees ~/Development/savera__worktrees
git -C ~/Development/savera worktree repair ~/Development/savera__worktrees/phase-0
git -C ~/Development/savera worktree list
```

`worktree repair` fixes the absolute paths in both directions: the worktree's `.git` file and the main repository's `.git/worktrees/phase-0/gitdir`. Workmux's git config keys (`workmux.worktree.phase-0.*` and `branch.feature/phase-0.workmux-base`) are keyed by handle and branch rather than path, so they still apply.

Rename the tmux session and window. The shells in each pane still point at the old directory, so `cd` each one to its new path:

```bash
tmux rename-session -t cellino savera
tmux rename-window -t savera:cellino savera
```

Move the Claude Code project directories so that `claude --resume` in the new paths lists the earlier sessions. The directory names start with a hyphen, which is why `mv` needs `--`:

```bash
cd ~/.claude/projects
mv -- -Users-ctm-Development-cellino -Users-ctm-Development-savera
mv -- -Users-ctm-Development-cellino--worktrees-phase-0 -Users-ctm-Development-savera--worktrees-phase-0
mv -- -Users-ctm-Development-cellino--worktrees-plan-phases -Users-ctm-Development-savera--worktrees-plan-phases
```

The transcripts still record the old working directory, so the earlier turns of a resumed session show old paths.

## Stage 4: bring phase-0 onto the renamed plan

Restart Claude Code in both windows from the new paths. In the phase-0 worktree, run `git merge main`. It fast-forwards `feature/phase-0` onto the renamed plan, so Phase 0's ADRs, `AGENTS.md`, and CI start under the new name. A resumed phase-0 session still has the old plan filename in its context, so point it at `docs/plans/2026-09-13-savera-build-plan.md`.

## Follow-ups

These are outside this plan.

- Before v0.1.0, search the USPTO (International Classes 9 and 15) and India's trademark registry for SAVERA by hand.
- Vendor prefix across the family. fosforo declares `com.catamount.fosforo` in `src/clap/plugin.zig`, and its CHANGELOG calls that id permanent. springer records `com.catamount.springer` in `AGENTS.md` and its build plan. Neither project has a tag or release, so moving them to `com.catamountaudio` would break no saved host project, but each is its own change in its own repository.
- The AU manufacturer name stays `Catamount` in all three projects. If the vendor's display name should become "Catamount Audio" as well, change it in all three at once.

## Verification

Repository, on `main` after the merge:

- `git grep -n -i -E 'cellino|clno' -- ':!docs/plans/done/2026-09-14-rename-project-to-savera.md'` prints nothing.
- `git grep -n 'com\.catamount\.' -- ':!docs/plans/done/2026-09-14-rename-project-to-savera.md'` prints nothing, and `git grep -c 'com\.catamountaudio\.savera' -- docs/plans/2026-09-13-savera-build-plan.md` reports 2 lines.
- `git grep -c -i savera -- README.md docs/design docs/plans/2026-09-13-savera-build-plan.md` reports 1, 1, and 30 lines, matching the old counts.
- `git grep -c -E 'Svra|SVRA' -- docs/plans/2026-09-13-savera-build-plan.md` reports 5 lines: 3 for the subtype and 2 for the magic.
- `git log --follow --oneline docs/plans/2026-09-13-savera-build-plan.md` goes back to `7081c6f`.
- `git diff 5c9c603 -- docs/design` shows exactly one changed line.
- `prettier --check README.md docs/plans/2026-09-13-savera-build-plan.md` passes.
- `markdownlint-cli2 README.md docs/plans/2026-09-13-savera-build-plan.md` reports the build plan's 290 MD013 findings, on the same lines as before the rename, and nothing for `README.md`.
- `typos` is clean.

GitHub:

- `gh repo view cboone/savera --json name,isArchived,url` shows `savera`, not archived.
- `gh api repos/cboone/cellino --jq .full_name` prints `cboone/savera`, which confirms the redirect.
- `git ls-remote origin refs/heads/main` matches `git rev-parse main`.

Local:

- `git -C ~/Development/savera worktree list` shows both worktrees under `savera` paths, neither marked `prunable`, and `git -C ~/Development/savera__worktrees/phase-0 status` succeeds.
- `workmux list`, run in `~/Development/savera`, shows `phase-0`.
- No `cellino` entries remain in `~/Development` or `~/.claude/projects`, and `tmux ls` shows `savera`.
- `claude --resume` in `~/Development/savera` lists this session.

The repo has no code or build yet, so there is no build or test step to run.
