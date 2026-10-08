---
name: worklist
description: >-
  Show what is left to do on the current task as a stable, numbered list —
  grouped by who has to act (us / another team / the owner) and refreshed
  against git and the task's notes. IDs never change, so "do FE-07" means the
  same thing next week. Takes an optional `pl` or `en` flag for the language of
  the output, and an optional slug. Use when asked "what's left", "co nam
  zostało", "na czym stoimy z listą", "what can I start now", or when a
  long-running task has drifted and needs re-grounding. Invoke it yourself
  before proposing an order of work on a task that has run for more than a few
  sessions.
---

# worklist

A long task loses its shape. `/park` and `/resume` carry prose; this carries the
**work items**, each with a stable id, an owner, a status, and a way to check
whether it is still true.

The list is the file. Every invocation re-grounds it against reality and prints
it; the file is the only place ids live.

## Arguments

`$ARGUMENTS` holds up to two things, in any order, space-separated:

- **a language flag** — `pl` or `en`
- **a slug** — anything else

So `/worklist pl`, `/worklist vc-funds`, `/worklist pl vc-funds` and
`/worklist vc-funds en` are all valid. Two language flags, or two non-flag
words, is a typo — say which you used and carry on rather than guessing.

### Language

The flag changes **the printed output only. The file stays in English.** The rows
carry anchors, ids, paths and commit SHAs, and the file is a handoff artifact
another session or another person reads — translating its contents would break
greps and split the vocabulary in two.

- Resolution: explicit flag → `lang:` in the file header → `en`.
- An explicit flag **updates `lang:`**, so `/worklist` on its own keeps speaking
  whatever was last asked for.
- Under `pl`, translate the group headings, the counts line and the state line.
  **Leave verbatim:** ids, commit SHAs, file paths, endpoint paths, code
  identifiers, anchors, and any quoted literal such as `:paginateBy="null"`.
  Item text is prose and gets translated; a term with no natural Polish form
  stays English rather than being invented.

Headings under `pl`:

```
Do zrobienia teraz (3)
Zablokowane (10) — czeka na decyzję
Czeka na innych (10)
Zaparkowane (5) — nie zaczynaj bez pytania
Zamknięte: …
Stan: <branch> @ <sha>, drzewo czyste, 0 ahead, 28 pozycji otwartych
```

## Slug

Resolution order, stop at the first hit:

1. **Explicit arg** → `~/.claude/worklists/<arg>.md`.
2. **A worklist whose `repo:` matches the current git root** (`git rev-parse
   --show-toplevel`) **and whose `branch:` matches the current branch.** Read the
   header of each `~/.claude/worklists/*.md` and match on both.
3. **A worklist whose `branch:` matches the current branch**, when exactly one
   does. This is what makes a slug that deliberately differs from the branch keep
   working (a task may be tracked under `vc-funds` while the branch is
   `APP-5251-matchmaking-mvp`).
4. **Branch-derived** — `git branch --show-current`, last path segment,
   lowercased, spaces and slashes → `-`.

No match and no arg → say so and offer to create one; do not invent a slug.

**Repo beats branch, because one task spans two repos.** Full-stack work usually
gives the front end and the back end the *same* branch name, so branch alone is
ambiguous the moment a task is tracked as two lists. Step 2 is what disambiguates;
step 3 only fires when a single worklist claims the branch.

If several worklists match the branch and **none** matches the current repo — you
are in a third checkout, or a `repo:` path went stale — do not pick one. Name the
candidates with their `repo:` lines and ask. Guessing here writes ids into the
wrong list, and ids are the one thing that must not move.

### Paired worklists

Two lists tracking one task across two repos are **siblings**: each carries a
`sibling: ~/.claude/worklists/<other>.md` header, and they share ONE id space —
`FE-07` means the same item in both files.

- **Write your own file only. Never edit the sibling's** — not its rows, not its
  header, not even to be helpful. The owner asked for this explicitly on
  2026-08-21 after both sides had been reaching across. Two sessions editing one
  file is how a list loses an item nobody notices is gone.
- **Mint only your own prefix**, with one exception that has worked in practice:
  raising something FOR the other side means minting their prefix **in your own
  file** and letting them adopt it. `BE-10` was raised that way and the backend
  closed it under the same id.
- **`next-ids` is reconciled on READ, not by writing across.** Read both headers,
  take the higher of each counter, and write the result **only into your own
  file**. No number is ever reused on either side, so a counter that has drifted
  low is safe to raise and never safe to lower.
- **Mirror rows** carry the status `mirror` and are the sibling's items, kept for
  visibility. Read-only: never close a mirror on its own evidence, never reword
  it. When the owning list closes an item, move the mirror to `## Done` too, with
  the same evidence and the note `(mirror)`.
- Mirror only what gates or is gated across the boundary, plus whatever the other
  side has asked to watch. A mirror of everything is just a second copy.
- **Both files can be open in two live sessions at once.** Re-read immediately
  before writing and merge; a blind overwrite eats the other session's work.
- Print mirrors as their own group, after `parked`.

## File format

```markdown
# Worklist: <slug>

repo: /abs/path/to/repo
branch: <branch>
lang: en
sources:
  - ~/.claude/plans/<slug>-resume.md
  - /abs/path/to/some/backlog.md
prefixes:
  FE: this codebase, ours to write
  BE: another team or repo
  OWN: needs the owner's decision
next-ids: FE=09 BE=04 OWN=08

## Open

| id | status | item | anchor |
| --- | --- | --- | --- |
| FE-07 | open | fund's favourite star on the applicant row | `grep -rn is_favourite src/stores/orgVcApplicants.types.ts` |
| OWN-01 | blocked | what set of startups does the fund's tab browse? | backlog A1 |

## Done

| id | closed by | item |
| --- | --- | --- |
| FE-03 | `ea8176c5` | list from /vc-matches |
```

`prefixes` are per-worklist, not fixed by this skill — a task with no second team
needs no `BE`. Keep them to who must ACT; status carries everything else.

**Statuses:** `open` (startable now), `blocked` (needs a decision — say whose in
the item), `waiting` (someone else is acting), `parked` (do not start without
asking), `mirror` (a sibling worklist's item, read-only here), `done`.

## ID rules — the whole point

- **Never renumber. Never reuse.** `next-ids` only ever increases, including past
  ids whose items were deleted as no-longer-real.
- **Closing an item moves the row to `## Done` and keeps its id**, with the
  evidence that closed it (a commit SHA, "decided 2026-08-17", a deleted file).
- An item that turns out to be two items keeps its id for the part that matches
  the original wording and mints a new id for the rest. Do not silently widen an
  id's meaning — someone may have said "do FE-07" already.

## Refresh, every invocation

1. **Ground truth first, no guessing:**
   ```bash
   git branch --show-current
   git log -1 --format='%h %s'
   git status --short
   git rev-list --left-right --count @{upstream}...HEAD 2>/dev/null || echo "no upstream"
   ```
2. **Run each open item's `anchor`** where it is a command. An anchor that no
   longer matches is evidence the item is done; an anchor that matches when the
   item says `done` is evidence it regressed — say so loudly rather than
   silently flipping it back.
3. **Read the `sources`.** They are the prose; this is where new items come from
   and where a `blocked` item learns it was answered. Sources outside the current
   repo may be read-only — respect that, never edit them from here.
4. **Reconcile, then report the diff** — what closed, what appeared, what moved.
   A refresh that changes nothing should say "nothing moved", not reprint
   silently.
5. **Write the file back**, then print.

Items are only marked `done` on evidence. No evidence → leave it open and say
what you would need to close it.

## Output

**Markdown tables, one per group. This shape is confirmed by the owner
(2026-08-21) — do not fall back to an indented list.** A worklist is scanned, not
read, and a table is what makes the third column carry its weight.

Lead with what is startable, because that is the question being asked. Bold every
id: they are the handle people speak in.

```markdown
## Startable now (0)

Nothing on our side. Everything left waits on a decision or on the other team.

## Blocked — needs a decision (7)

| id | what has to be settled | source |
| --- | --- | --- |
| **OWN-02** | which endpoint scores the fund→startups direction, and its tie-break? | backlog A2 |
| **OWN-03** | does `computeMatch.ts` stay as a fallback once the server sends `match`? | backlog A3, gated by BE-05 |

## Waiting on others (2)

| id | what | waiting on |
| --- | --- | --- |
| **FE-13** | delete the per-page Match % banner — must die with server-side sorting | BE-05 |

## Parked — do not start without asking (4)

| id | what | state |
| --- | --- | --- |
| **FE-11** | six inlined copies of the intro/message/rate strip | **unblocked** by the OWN-01 decision |

## Mirrors from the sibling list (6)

| id | what | their state |
| --- | --- | --- |
| **BE-05** | phase 8 — server-side match score, designed, not built | `open` · gates FE-13 and OWN-03 |

## Closed (20)

**With a commit:** FE-01 `964862fc` · FE-02 `ea8176c5` · …

**Closed on other evidence:** FE-07 guide + xlsx + artifact · FE-09 measured, no
code needed · FE-19 cited in BE-04

**The other side's:** BE-03 `3d9412ea` · BE-10 `25721a3e` · …

**State:** `<branch>` @ `<sha>`, tree clean, 0 ahead, 19 open items.
```

Rules the shape encodes:

- **Every row is printed in full. Never collapse a range** — no `OWN-02 … OWN-09`,
  no `...`, no "and 6 more". The owner asked for this explicitly after a collapsed
  range hid what `OWN-09` actually was.
- **The third column differs per group on purpose** — `source` for blocked (where
  the decision is written down), `waiting on` for waiting (which id or event
  unblocks it), `state` for parked (why it is parked, or that it is now unblocked),
  `their state` for mirrors (the sibling's status, verbatim).
- **An empty group gets a sentence, not an empty table.** `Startable now (0)`
  followed by one line saying what that means is information; an empty table is
  furniture.
- **`Closed` is three compact inline lists, not a table** — with a commit, closed on
  other evidence, the other side's. Items with no SHA are the ones a reader will
  otherwise hunt for, so they get their own bucket rather than being hidden. When
  something closed since the last refresh, name it in a line above the state line.
- Group order is fixed: startable, blocked, waiting, parked, mirrors. Within a
  group keep file order.
- If an item gates another, say so in the third column (`gates FE-11`). Dependency
  arrows are worth more than a priority column nobody maintains.

**Group STRICTLY by status, and never name a group after a prefix.** Prefix and
status are orthogonal on purpose — prefix is who acts, status is what state the
item is in — so a heading like "Waiting on BE" is a category error: an `FE` item
can be `waiting` (gated by someone else's work) and will silently fall out of
every group. This happened on the first real run: `FE-13` vanished because the
waiting group had been labelled by prefix. Name the owner per row, which the id
prefix already does, or after a dash on the heading when one owner genuinely
holds the whole group.

**Invariant, check it before printing:** the group counts must sum to the number
of rows in `## Open`. Print that total on the state line — `19 open items` — so a
dropped row is visible instead of plausible. If they disagree, say so and print
the unassigned ids rather than a tidy list that is missing something.

## Guardrails

- **This skill writes `~/.claude/worklists/<slug>.md`** and, when that file has a
  `sibling:`, the sibling's mirror rows and `next-ids` line — nothing else. It
  never edits the repo, never edits the `sources`, never commits.
- **Never invent items to look thorough.** Every row traces to a source, a
  measurement, or something the owner said. If the list feels short, it is short.
- **Do not re-litigate settled calls.** Decisions live in the resume note's
  *Decisions NOT to reopen*; an item that contradicts one of those is a mistake
  in the item.
- Keep item wording to one line and greppable. The reasoning belongs in the
  sources, not here.
