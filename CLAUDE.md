v001 | 2026-08-19 | 254 lines

# Working Rules

Process lives in four tiers, by ownership:

- **CLAUDE.md** — universal human/Claude working rules that apply to every
  project. Contains Part A (behavioral), Part B (code discipline), and Part C
  (shared processes).
- **SCOPE.md** — describes the project and lists the other governance files it
  requires. Each project declares its own type, required governance docs, and
  branch/PR discipline here. Read it first to know where the project sits.
- **XYZ.md** — project-specific files that apply only to that project, if
  applicable. Live in the repo.
- **Google Doc SOP** — processes done entirely by the human. A user reference,
  not a source of truth.

**Precedence.** SCOPE.md governs project-specific matters. Where SCOPE.md and
CLAUDE.md conflict, CLAUDE.md Part A and Part B govern. Part C is the exception:
SCOPE.md is what declares which Part C processes a project opts into, so where
SCOPE.md and Part C conflict, SCOPE.md governs.

## Part A — Behavioral (applies to all work)

**Read the governance files first.** Read SCOPE.md at the start of every task —
in a Claude chat, the project-folder copy; in Code, the version on
`origin/main`. Read CLAUDE.md as well when the task touches the repo,
governance, or the backlog: Part A is already pasted into my global settings, so
Parts B and C are what opening the file adds. Before any repo-touching work,
read the full set SCOPE.md lists. Read once per chat at the first task, not per
exchange. If a required governance file is missing, stop and ask before doing
any work.

**Don't guess.** If something is missing, ambiguous, or uncertain, stop and ask
— never guess at a file's contents, a convention, or my intent. When you agree
with a proposal of mine, restate it more concisely without drifting back toward
your own phrasing; if you still see a genuine problem with it, say so explicitly
rather than quietly changing the wording.

**Ask for content first.** If I say I'm sending content but nothing is attached,
assume I hit send before attaching it. Stop and ask for the content — do not
fill the gap with a speculative or elaborated response before I've provided it.

**Match the mode.** Not every chat is about coding. Do not default to
coding-oriented output unless the chat is actually about building or modifying
code. If unsure which mode we're in, ask before proceeding.

**Be concise and direct.** Keep responses focused. Push back when you disagree —
don't just agree to be agreeable. Do not over-explain your reasoning; I will ask
if I want more.

**Write American English.** Everything you author — chat, governance docs, code
comments, commit messages, user-facing strings — uses American spelling and
usage: `organize`, `analyze`, `color`, `licensed`, `-ize` over `-ise`. The
exception is names you don't own: an identifier, API, filename, or quoted
passage keeps the spelling it was given, and is never silently "corrected" to
match this rule.

**The repo is the source of truth.** In any project with a git repo, the
committed, merged repo (git main) is authoritative — not another conversation,
not memory, not an attached or synced copy. If attached or synced project files,
or claims about what was decided in another chat, conflict with the repo, flag
the conflict, stop, and ask before proceeding.

**Prompts are for Claude Code.** When I ask for a prompt to run, write it for
Code to execute — clear, scoped, and based on what we discussed. I will review
it before running. Always put it in a code block so I can copy-paste it
directly. If you amend a prompt already written, regenerate it in full.

**Discussion mode by default — wait for the go.** Do not write prompts, code, or
files until I explicitly say so (e.g. "write it now"). Until then, ask
questions, surface tradeoffs, and refine the thinking with me. Surfacing options
and recommending one is always fine; producing the deliverable waits for my
word, even if the discussion feels complete. Do not draft until all your own
questions are answered.

## Part B — Code discipline (projects with a git repo)

These rules apply to any project with a git repo, coding or not, per its
SCOPE.md. Clauses referencing `origin/main`, pushing, or PRs apply only where
the repo has a remote; the rest apply to local-only repos too.

**Read governance docs from `origin/main`.** Always `git fetch origin` first,
then read the committed version (`git show origin/main:<file>`) — never the
working tree, never the synced project folder.

**Sync audit.** At the start of every repo-touching task (pure discussion turns
don't need it), compare line 1 — version, date, line count — between
`origin/main` and the project-folder copy. Audit whatever governance docs
SCOPE.md lists, plus CLAUDE.md and SCOPE.md themselves, plus any stamped
non-`.md` governance file the project keeps (e.g. STYLE.css); never assume a
fixed list. Code performs the whole audit: it reads both sides itself — the
committed version from `origin/main` and the project-folder copy — compares
them, and reports the joined result. A chat cannot read git, so it is never
asked to supply half of it. Where the two differ, the higher version is
current: sync to it and report what you did, without waiting to be told.

**Governance docs go direct to main.** Every `.md` governance doc listed in
SCOPE.md — CLAUDE.md, SCOPE.md, BLOG.md, BACKLOG.md, PROCESS.md and the like —
is committed straight to main and pushed immediately. No branch, no PR. Their
blast radius is small: they carry no styling or page logic, so a bad edit
degrades a doc or an internal view, never the public site. Push is the finish
line: an edit left uncommitted or unpushed is the failure this rule exists to
prevent.

**Publicly-served governance files are the exception.** Where SCOPE.md declares
a governance file publicly served, an authored change to it keeps branch and PR
discipline: committing straight to main would change the live site with no
review. A byte-identical copy of content already reviewed in the repo where it
was authored is not an authored change, and goes direct to main like any other
governance doc.

**Governance docs carry a version stamp.** Line 1 of every `.md` governance doc
is exactly `v### | yyyy-mm-dd | #### lines` — nothing above it, no title, no
blank line. A file whose format forbids bare text at the top (e.g. STYLE.css,
which must open with a comment) carries the identical stamp on the first line of
its opening comment block.

**Every field of the stamp is machine-generated, never typed.** Stamp last,
once the content edit is final, in this order:

1. `v###` — read the existing stamp and add one, zero-padded to three digits
   (`printf 'v%03d'`). A new doc starts at `v001`. Bump on every substantive
   change. Counters are per-file and independent: each doc's `v###` tracks its
   own history, so a constant offset between two files is expected and is not a
   defect to reconcile.
2. `yyyy-mm-dd` — `date +%F`, read in the machine's local timezone. The stamps
   are for the human reading them, so a doc edited late at night carries that
   night's date.
3. `#### lines` — `grep -c '' <file>`, not `wc -l`. `wc -l` counts newlines, so
   it undercounts by one on a file with no trailing newline. Take the count with
   the stamp line already in place, so the number includes itself; write a
   placeholder line 1 first on a new doc, then count, then overwrite it.

A hand-typed date is how a doc ends up stamped ahead of the actual day, and a
hand-typed line count is worse than none. If machine time returns something
implausible, stop and ask rather than stamp it.

**Protect main when it deploys from main.** If the project deploys from main (a
live website or app), never commit code directly to main: one feature branch per
task, branched from an up-to-date main → commit locally as you work → push the
branch → open a PR only when I ask → merge → clean up. If the project does not
deploy from main, committing code directly to main is fine; branch only when you
want isolation for risky work.

**No PR unless I explicitly ask.** When I do ask, name it
`type/short-description`, where type is one of: feat, fix, docs, refactor,
chore, style, test, perf, build, ci, uat.

**Post-merge cleanup — Claude reminds, so I don't have to.** After any PR
merges, Claude surfaces the cleanup automatically, but only once the merge is
confirmed on main (a merge commit in `git log`, or my confirmation). The
sequence is push → PR → merge → cleanup, and cleanup is gated on the merge being
confirmed. Once it is, provide: `git checkout main`, `git pull origin main`,
`git branch -d <branch>`, `git remote prune origin`.

**Style changes get their own commit.** When a UI/UX decision is finalized and
applies project-wide (not a one-off), ask whether STYLE.md and/or STYLE.css
should be created or updated. Any change to either gets its own commit, never
mixed into other changes.

**Show client-facing changes on localhost.** When edits are client-facing UI or
UX changes (style, content, layout, flow), launch localhost first so I can see
them before they're committed. Start the server with caching disabled, or hand
over the URL cache-busted (`?v=<short-sha>` or equivalent) — never a bare
`localhost:####`. Beating the cache is Claude's job, not mine: I should never
have to hard-reload, clear a cache, or open a private window to see the version
you just built, and a review of a stale page is a wasted review.

## Part C — Global Human/Claude Processes

Processes here are either **universal** — they apply to every project, always —
or **opt-in**, applying only where SCOPE.md declares them.

### Environment (universal)

- **Disk master.** `/Users/Stella/spwgit/CLAUDE.md` — the master copy every
  project's CLAUDE.md reconciles against.
- **GitHub account.** `https://github.com/Stellapwang` — every project remote
  lives under this account. The specific remote URL belongs in that project's
  SCOPE.md, not here.

### how-to: keep CLAUDE.md in sync (universal)

The master copy of CLAUDE.md lives on local disk at the path above. Every
project repo holds a copy, and the disk master is what they reconcile against.

**Publishing a change (project A).** Edit the copy in project A's repo, commit
to main, then copy the file back to the disk master. The change is not finished
until both have happened: an uncommitted edit is invisible to other projects,
and a stale disk master is a file every other project will sync backwards from.

**Reconciling a copy (any project).** At the start of a task, compare the
project's copy against the disk master by line 1, and where they differ, **sync
to the later version automatically** — Claude performs the sync and reports it,
rather than flagging the gap and waiting to be told. The comparison is
two-sided, so it has two outcomes. **Disk master later:** copy it into the
working tree, commit, and push — files move disk → working tree → commit → push,
in that order, since a file cannot enter git any other way. **Repo copy later:**
copy it out to the disk master, which is the publishing step above run on its
own. Either way both copies end the task on the same version, and a gap is never
carried forward across tasks. In a project with no git repo, the same
reconciliation runs between the disk master and the project's copy; the commit
and push steps simply don't arise.

**Human steps after any change.** Re-paste Part A into Claude's global settings,
and upload the current file to each Claude project folder.

### how-to: maintain the backlog (opt-in — when SCOPE.md declares it)

The backlog tracks both short-term items (bugs, UI improvements) and long-term
ones (big features, new apps). The definitions — categories, status semantics —
live in the BACKLOG.md header.

**Process vs. artifact.** The process here is global and identical everywhere.
The artifact is per-project: each project keeps its own `BACKLOG.md` in its own
repo. One shared process, one backlog file per project; they never merge across
projects.

**Two states.** The distinction is the whole system:

- **The running block** — a live tally *in chat*. Temporary, uncommitted, holds
  items as they are raised during a session.
- **BACKLOG.md** — the source of truth *in the repo*.

Items flow one direction: raised in chat → held in the running block → flushed
to BACKLOG.md.

**The running block.** The trigger is my saying "log to backlog." The block
reprints in full every time it changes — never the new row alone — so the latest
printing is always the complete, authoritative list. Temp IDs are `P01`, `P02`,
… scoped to the current unflushed batch only; after a flush the block empties
and they recycle from P01.

**Flushing.** I request the flush; nothing flushes automatically. It is a
word-for-word copy verified by count — N pending rows in the block = N new rows
out — which guards against dropped or duplicated rows. P## become permanent
`BL-###`, assigned in cumulative sequence from the last BL number in the file
and never reused. Category and status are assigned at flush. BACKLOG.md is a
governance doc, so the flush is committed and pushed per Part B. If the project
publishes a rendered backlog view, verify the flush there rather than in the raw
file: push, wait for the deploy to land, then confirm the new rows render as
correctly-columned rows. This is what catches an unescaped pipe, which is silent
in the source and only visible once rendered. Verify against the deployed page,
not a local copy, and not before the deploy confirms.

**Status rules.** Code never self-closes: done items move to `review`, not
`close`. Close is mine alone and needs evidence in **Closed-by** — the `PR##`
for code, or my stated reason otherwise. A `discard` row carries its reason
there too: a discard without a recorded reason is a vanished row. Closed-by
stays empty on every other status.

**Schema.** `| ID | Status | Category | Item | Raised | Closed-by |` — the
running block uses the same columns, with a `P##` in the ID slot.
