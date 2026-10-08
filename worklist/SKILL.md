---
name: worklist
description: >-
  Show what is left to do on the current task as a stable, numbered list,
  grouped by who has to act (us / another team / the owner) and refreshed
  against git and the task's notes. IDs never change, so "do DEV-07" means the
  same thing next week. Takes an optional language flag (e.g. `en`, `pl`, `de`)
  for the language of the output, and an optional slug. Use when asked "what's
  left", "where do we stand on the list", "what can I start now", or when a
  long-running task has drifted and needs re-grounding. Invoke it yourself
  before proposing an order of work on a task that has run for more than a few
  sessions.
---

# worklist

A long task loses its shape. Prose handoff notes carry the story; this carries
the **work items**, each with a stable id, an owner, a status, and a way to
check whether it is still true.

The list is the file. Every invocation re-grounds it against reality and prints
it; the file is the only place ids live.

## Arguments

`$ARGUMENTS` holds up to two things, in any order, space-separated:

- **a language flag**: a two-letter language code such as `en`, `pl`, `de`
- **a slug**: anything else

So `/worklist de`, `/worklist billing`, `/worklist de billing` and
`/worklist billing en` are all valid. Two language flags, or two non-flag
words, is a typo. Say which you used and carry on rather than guessing.

### Language

The flag changes **the printed output only. The file stays in English.** The rows
carry anchors, ids, paths and commit SHAs, and the file is a handoff artifact
another session or another person reads. Translating its contents would break
greps and split the vocabulary in two.

- Resolution: explicit flag → `lang:` in the file header → `en`.
- An explicit flag **updates `lang:`**, so `/worklist` on its own keeps speaking
  whatever was last asked for.
- When the language is not `en`, translate the group headings, the counts line
  and the state line. **Leave verbatim:** ids, commit SHAs, file paths, endpoint
  paths, code identifiers, anchors, and any quoted literal such as
  `--feature-flag=false`. Item text is prose and gets translated; a term with no
  natural equivalent stays in English rather than being invented.

## Slug

Resolution order, stop at the first hit:

1. **Explicit arg** → `~/.claude/worklists/<arg>.md`.
2. **A worklist whose `repo:` matches the current git root** (`git rev-parse
   --show-toplevel`) **and whose `branch:` matches the current branch.** Read the
   header of each `~/.claude/worklists/*.md` and match on both.
3. **A worklist whose `branch:` matches the current branch**, when exactly one
   does. This is what makes a slug that deliberately differs from the branch keep
   working (a task may be tracked under `billing` while the branch is
   `PROJ-1234-billing-revamp`).
4. **Branch-derived**: `git branch --show-current`, last path segment,
   lowercased, spaces and slashes → `-`.

No match and no arg → say so and offer to create one; do not invent a slug.

**Repo beats branch, because one task can span two repos.** Full-stack work often
gives the front end and the back end the *same* branch name, so branch alone is
ambiguous the moment a task is tracked as two lists. Step 2 is what disambiguates;
step 3 only fires when a single worklist claims the branch.

If several worklists match the branch and **none** matches the current repo (you
are in a third checkout, or a `repo:` path went stale), do not pick one. Name the
candidates with their `repo:` lines and ask. Guessing here writes ids into the
wrong list, and ids are the one thing that must not move.

### Paired worklists

Two lists tracking one task across two repos are **siblings**: each carries a
`sibling: ~/.claude/worklists/<other>.md` header, and they share ONE id space.
`DEV-07` means the same item in both files.

- **Write your own file only. Never edit the sibling's**: not its rows, not its
  header, not even to be helpful. Two sessions editing one file is how a list
  loses an item nobody notices is gone.
- **Mint only your own prefix**, with one exception: raising something FOR the
  other side means minting their prefix **in your own file** and letting them
  adopt it. When they close it, they close it under the same id.
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
  DEV: this codebase, ours to write
  EXT: another team or repo
  OWN: needs the owner's decision
next-ids: DEV=09 EXT=04 OWN=08

## Open

| id | status | item | anchor |
| --- | --- | --- | --- |
| DEV-07 | open | add pagination to the users list endpoint | `grep -rn "page_size" src/api/users.ts` |
| OWN-01 | blocked | which environments does the nightly job run against? | backlog A1 |

## Done

| id | closed by | item |
| --- | --- | --- |
| DEV-03 | `abc12345` | extract shared date helpers |
```

`prefixes` are per-worklist, not fixed by this skill. A task with no second team
needs no `EXT`. Keep them to who must ACT; status carries everything else.

**Statuses:** `open` (startable now), `blocked` (needs a decision; say whose in
the item), `waiting` (someone else is acting), `parked` (do not start without
asking), `mirror` (a sibling worklist's item, read-only here), `done`.

## ID rules: the whole point

- **Never renumber. Never reuse.** `next-ids` only ever increases, including past
  ids whose items were deleted as no-longer-real.
- **Closing an item moves the row to `## Done` and keeps its id**, with the
  evidence that closed it (a commit SHA, "decided 2026-01-15", a deleted file).
- An item that turns out to be two items keeps its id for the part that matches
  the original wording and mints a new id for the rest. Do not silently widen an
  id's meaning; someone may have said "do DEV-07" already.

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
   item says `done` is evidence it regressed. Say so loudly rather than
   silently flipping it back.
3. **Read the `sources`.** They are the prose; this is where new items come from
   and where a `blocked` item learns it was answered. Sources outside the current
   repo may be read-only. Respect that, and never edit them from here.
4. **Reconcile, then report the diff**: what closed, what appeared, what moved.
   A refresh that changes nothing should say "nothing moved", not reprint
   silently.
5. **Write the file back**, then print.

Items are only marked `done` on evidence. No evidence → leave it open and say
what you would need to close it.

## Output

**Markdown tables, one per group. Do not fall back to an indented list.** A
worklist is scanned, not read, and a table is what makes the third column carry
its weight.

Lead with what is startable, because that is the question being asked. Bold every
id: they are the handle people speak in.

```markdown
## Startable now (0)

Nothing on our side. Everything left waits on a decision or on the other team.

## Blocked: needs a decision (2)

| id | what has to be settled | source |
| --- | --- | --- |
| **OWN-02** | which endpoint serves the summary view, and what is its tie-break? | backlog A2 |
| **OWN-03** | does the client-side fallback stay once the server sends the value? | backlog A3, gated by EXT-05 |

## Waiting on others (1)

| id | what | waiting on |
| --- | --- | --- |
| **DEV-13** | delete the client-side banner once sorting moves to the server | EXT-05 |

## Parked: do not start without asking (1)

| id | what | state |
| --- | --- | --- |
| **DEV-11** | deduplicate the inlined copies of the header component | **unblocked** by the OWN-01 decision |

## Mirrors from the sibling list (1)

| id | what | their state |
| --- | --- | --- |
| **EXT-05** | server-side scoring, designed, not built | `open` · gates DEV-13 and OWN-03 |

## Closed (5)

**With a commit:** DEV-01 `abc12345` · DEV-02 `def67890` · …

**Closed on other evidence:** DEV-07 guide published · DEV-09 measured, no
code needed

**The other side's:** EXT-03 `0a1b2c3d` · …

**State:** `<branch>` @ `<sha>`, tree clean, 0 ahead, 5 open items.
```

Rules the shape encodes:

- **Every row is printed in full. Never collapse a range**: no `OWN-02 … OWN-09`,
  no `...`, no "and 6 more". A collapsed range hides what the middle items
  actually are.
- **The third column differs per group on purpose**: `source` for blocked (where
  the decision is written down), `waiting on` for waiting (which id or event
  unblocks it), `state` for parked (why it is parked, or that it is now unblocked),
  `their state` for mirrors (the sibling's status, verbatim).
- **An empty group gets a sentence, not an empty table.** `Startable now (0)`
  followed by one line saying what that means is information; an empty table is
  furniture.
- **`Closed` is three compact inline lists, not a table**: with a commit, closed on
  other evidence, the other side's. Items with no SHA are the ones a reader will
  otherwise hunt for, so they get their own bucket rather than being hidden. When
  something closed since the last refresh, name it in a line above the state line.
- Group order is fixed: startable, blocked, waiting, parked, mirrors. Within a
  group keep file order.
- If an item gates another, say so in the third column (`gates DEV-11`). Dependency
  arrows are worth more than a priority column nobody maintains.

**Group STRICTLY by status, and never name a group after a prefix.** Prefix and
status are orthogonal on purpose (prefix is who acts, status is what state the
item is in), so a heading like "Waiting on EXT" is a category error: a `DEV` item
can be `waiting` (gated by someone else's work) and will silently fall out of
every group. Name the owner per row, which the id prefix already does, or after
a dash on the heading when one owner genuinely holds the whole group.

**Invariant, check it before printing:** the group counts must sum to the number
of rows in `## Open`. Print that total on the state line (`19 open items`) so a
dropped row is visible instead of plausible. If they disagree, say so and print
the unassigned ids rather than a tidy list that is missing something.

## Guardrails

- **This skill writes `~/.claude/worklists/<slug>.md` and nothing else.** It
  never edits the sibling worklist, never edits the repo, never edits the
  `sources`, never commits.
- **Never invent items to look thorough.** Every row traces to a source, a
  measurement, or something the owner said. If the list feels short, it is short.
- **Do not re-litigate settled calls.** Decisions recorded in the sources as
  settled (for example under a "Decisions not to reopen" heading in a resume
  note) stay settled; an item that contradicts one of those is a mistake in the
  item.
- Keep item wording to one line and greppable. The reasoning belongs in the
  sources, not here.
