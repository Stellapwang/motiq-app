v003 | 2026-09-04 | 95 lines

# SCOPE — motiq-app

## What this is

The MotiQ pilot web app: a single self-contained HTML file that runs the drawing and
tapping battery on iPad. It is the prototype instrument used for feasibility work, not
the production instrument. The native iPadOS build is a separate future project.

## Repo and deploy

- Remote: https://github.com/Stellapwang/motiq-app
- Deploy target: Netlify project `motiq-pilot-v01` -> https://motiq-pilot-v01.netlify.app
- **Production deploys from `main`.** Whatever is on main is what participants run.
- **Branch deploys are enabled.** Every pushed branch gets its own Netlify URL, for
  testing only. A branch URL is the only way to test the voice conditions on the iPad:
  `getUserMedia` is exposed only on a secure origin, so the app opened from a Mac's LAN
  address over plain http looks normal and records nothing.
- **Never deploy by hand.** A manual upload produces a live file that corresponds to no
  commit, which is how `v9.02` came to exist nowhere but on Netlify.
- Build command: none. Publish directory: repo root. The deployed artifact is
  `index.html` and nothing else; the app has no external assets and loads nothing from
  a CDN.

## Governance files required

- CLAUDE.md
- SCOPE.md
- BACKLOG.md

No governance file in this repo is publicly served. `index.html` is application code, not
a governance doc, and carries no version stamp.

## Branch and PR discipline

Because main deploys, CLAUDE.md Part B applies in full: never commit application code
directly to main. One feature branch per task, branched from an up-to-date main, commit
locally, push the branch, PR only when asked, merge, then clean up.

SCOPE.md, CLAUDE.md, and BACKLOG.md are governance docs and go direct to main.

## Releases and data provenance

The app writes its own version into every exported record, alongside the export schema:

    schema:"motiq_prototype/6", build:BUILD

That makes `BUILD` the provenance key linking collected pilot data to the code that
produced it. It cannot be reconstructed after the fact.

Therefore:

- Every release is tagged, and **the tag string equals the version field of `BUILD`
  exactly** (`v9.02`, not `v9` or `v09`). Given a CSV from a session, `git checkout <build>`
  must return the exact code that ran it.
- **Any build used to collect data is tagged before the session, including
  self-testing.** An untagged build does not identify unique code: `BUILD` stays
  constant across every commit on a branch, so a record naming an untagged `BUILD`
  cannot be resolved to the code that produced it.
- Bumping `BUILD` is the last edit before a release merges, and the tag is applied to the
  merge commit on main.
- Filenames are not version identifiers. The pre-git history contains files whose names
  disagree with their `BUILD` (`index_v01.html` reports `v0.3`). `BUILD` governs.

## Version history before git

Tags `v0.3` through `v8.02` were imported on 2026-08-26 from files kept outside version
control. Their commit dates are the import date, not the date the work was done; each
commit message records the original build date. The history is reconstructed and is
labeled as such — it is not a record of when the changes were made.

Only versions that were hardened and deployable were imported. Intermediate chat drafts
(`motiq_webapp_*.html`) were deliberately left out of the repo and remain in iCloud.

`v9.02` is the exception worth remembering: it was live on Netlify but existed nowhere
else, because the deployed file was never saved back. It was recovered by downloading the
deploy. This repo exists so that cannot happen again.

`v11.01` is the second provenance failure this repo has recorded. It was deployed to
production from an unmerged branch while SCOPE.md still declared that production
deployed from `main`, and it was used to collect data on 2026-09-03 without a tag.
The branch was merged and tagged afterwards, so the code survives; which of the
branch's commits produced any given 2026-09-03 record does not.

## Backlog

Adopted. This project uses the Part C backlog process, with its own `BACKLOG.md` in this
repo. Categories and status semantics are defined in that file's header.

## Data rules

Never commit participant data, PHI, licensed instrument content, or unpublished datasets
to this repo. The app stores trial data in browser storage on the device; exports stay off
this repo entirely.
