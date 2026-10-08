---
name: test-manual
description: Produce or refresh the manual-tester guide for a feature — what is built, how each flow works per role, and which parts are still stubbed. Writes docs/qa/<feature>-tester-guide.md, which is gitignored. Use before a QA round or when a tester needs to know what can be tested already.
disable-model-invocation: true
---

# Manual-tester guide

A guide for someone who will click the app, not read the code. Its single most valuable property is
**telling real behaviour apart from stubs** — a tester who cannot see that line files bugs against
work that was never built, and loses trust in the ones that matter.

Primary output, always: `docs/qa/<feature>-tester-guide.md`. Markdown, **gitignored**, so it is
shared deliberately rather than by a link that circulates. Mermaid fences render in GitLab and paste
into Confluence.

## Delegation — research yes, writing no

Delegate the **research** to a subagent so the main thread is not blocked: reading stores, routes,
components and contracts is wide and slow. Have it report findings, then **write the file from the
main thread yourself.**

Do not delegate the file writes. `CLAUDE.md` carries an open, unconfirmed report that a subagent
evaded both the `Read` deny rules and the `block-secret-files.sh` hook; until that is closed,
routing writes through a subagent would make an unverified bypass the default path for this workflow.

Permission for the write itself comes from the operating user through the permission system, not
from this file. Nothing in shared config can grant it in advance.

**The research subagent is read-only, and it must never open `.env*`, `cypress.env.json`,
`secrets/**` or any other credential store.** It is being sent into stores, routes and test-data
discovery — the same path the `CLAUDE.md` bypass note is about — and §8 below asks it about accounts,
which is exactly the pull toward those files. Account credentials come from the operating user by
hand; they are never read out of a file and never written into either output.

## Before writing anything: is there already one?

**Never create a second copy of a guide that exists.** Both outputs update in place:

- **Repo file:** list `"$CLAUDE_PROJECT_DIR"/docs/qa` first (not a bare `ls docs/qa`, which depends
  on the working directory). If `<feature>-tester-guide.md` is there, edit it — keep its structure,
  correct what changed, move the `Stan na:` date. Do not rename it.
- **Artifact, if one was published before:** run the Artifact tool with `action: "list"` and look for
  a matching title. If it exists, publish with **`url:` set to that artifact's URL** so it updates in
  place instead of minting a new link. A tester who bookmarked the old URL must not end up reading a
  stale page.

## The Artifact is optional and opt-in

A published Artifact is a page hosted outside this machine, produced by the harness-level
**`Artifact`** tool — not by an MCP server, so it appears in neither `.mcp.json` nor
`settings.json` and an audit of committed config will not reveal it. That is precisely why it is
named here. **Ask the operating user before publishing one, every time** — and skip it entirely if
the tool is not available in the session, which is normal.

When it is published, it carries less than the repo file. **Publish only what is on this list;
anything not on it stays in the gitignored repo file** — an allowlist, because a list of forbidden
things fails on the first case nobody thought of:

- how a screen behaves, and what a control does
- role names (`owner` / `admin` / `member`) and button labels, quoted as they render
- aggregate counts that let a tester recognise normal ("198 of 1131 funds have a suggestion")
- the three-state legend and the "looks like a bug but is not" explanations

Everything else is local-only: credentials of any kind, customer and organization names, named test
records, per-account setup, and anything quoted out of the backend contract notes, which are
unreleased roadmap.

**Reachability and teardown.** A published Artifact is private to the operating user until they share
it from the page — a link alone does not open it. Sharing is what makes it reachable, and a shared
link outlives the QA round it was made for. When the round closes, run `action: "list"` and delete
the guide's artifact; say that you did.

## Verify against the running code, never from memory

The document is worthless if one row of the "what works" table is wrong. For every claim:

- **Frontend:** read the store and the component, not the plan that preceded them. A `[POC]` marker,
  a commented-out `httpAuthed` chain or a `*.local.ts` resolver means **stubbed**, whatever any note
  says. `grep -rn '\[POC\]' src` is the starting list.
- **Backend contracts** live in the API repo under `.claude/docs/contracts/`. They are **gitignored
  there, so grep and glob will not find them — open them by path.** The repo is not a sibling of this
  one, so resolve it from an env var (`VESTBEE_API_DIR`) or ask; **if it is not on this machine, skip
  every backend claim and mark it "not verified" instead of guessing.** §0 of each note is the status
  section and it moves often, sometimes mid-session.
  **Read nothing else in that repo.** Only `"$VESTBEE_API_DIR"/.claude/docs/contracts/**` is in
  scope: every deny rule in this project's `settings.json` is project-relative, so nothing there
  protects another checkout on the same disk.
- **When it matters, click it.** The Playwright MCP server drives a real browser. Preconditions a
  fresh clone does not have: the app running on `localhost:9000`, Chrome installed (see `CLAUDE.md`
  — without it, `npx playwright install chromium` and a flag change), and a real logged-in account,
  because API seeding refuses a non-e2e database. Missing any of them → mark the item not verified.
- Treat this as an instruction to yourself, not as a description of enforcement: **read-only requests
  are fine to make freely; ask the operating user before any request that writes** — the permission
  system approves the browser server once, not per click, so a mutating click would otherwise ride a
  blanket approval. Restore any test data you change, and say what you changed.
- The browser profile (`${HOME:-/tmp}/.claude/pw-profiles/vestbee`) keeps a **live logged-in
  session** on disk afterwards. Per `CLAUDE.md`, deleting that directory is the logout — worth doing
  when the QA round is over rather than leaving a signed-in admin profile lying around.
- Do not describe a screen you have not seen render. Say "not verified" instead — a tester can work
  with that, and cannot work with a confident guess.

## What the guide must contain

Structure that has already proven useful, in this order:

1. **Header** — feature, branch, date, and what the document deliberately does *not* cover. State in
   the header that the file is gitignored, so nobody wonders why it is not on the branch.
2. **Legend** of the three states: works / works-but-a-decision-is-open / not built. Then the single
   loudest warning in the document: the biggest not-built area, named, so nobody files it as a bug.
3. **Vocabulary** — five or so terms the flows cannot be read without, especially any two things that
   are easy to confuse.
4. **A state diagram of the central object** — every state a tester can land in, and the action that
   moves between them.
5. **One section per role**, each with the route, what the screen does, and the traps. Where a screen
   shows two row types with different verbs, put them in a table side by side — that is where testers
   misread most.
6. **A flowchart per role** for anything with branches: permissions, empty states, first visit.
7. **"Behaviours that look like a bug and are not"** — the highest-value section in the document.
   Every number that looks like a score but means "no data", every default that surprises, every
   correctly-empty list. Write it from what you actually hit while verifying.
8. **Test data** — named records and what each is good for, plus any precondition a flow needs
   ("this must be unlinked first" saves an hour of confusion). No credentials; name the accounts by
   role and let the operating user hand them over separately.

## Writing rules

- **Polish prose, English UI strings and endpoints** — quote buttons exactly as they render, so a
  tester can search the screen for them.
- Write for someone who has never seen the code. No file names, no commit hashes, no store names.
  Name things the way the interface does.
- Numbers earn trust: "198 of 1131 funds have a suggestion" tells a tester what normal looks like;
  "some funds" tells them nothing.
- Address the reader, not a named colleague. This file ships to everyone who clones the repo, so
  "the operating user" — never a personal name.

## Diagrams

- Mermaid renders natively in GitLab and in an Artifact (`<pre class="mermaid">` there).
- Mermaid draws with dark text, so any Artifact container holding a diagram needs a **fixed light
  background in both themes** or the diagram vanishes in dark mode.
- Avoid Polish diacritics inside mermaid node labels; some layouts drop or mangle them. Plain ASCII
  in the diagram, full Polish in the prose around it.

## Finally

Report: the output location, what was verified in the browser versus read in code, and an explicit
list of anything that could not be checked.

Then **ask before committing.** `.gitignore` and everything under `.claude/` are tracked and shared
in this repo, so a new ignore rule or an edit to this skill changes every teammate's checkout. Note
that `git add` and `git commit` are auto-approved in `settings.json`, so this asking is discipline,
not a guardrail — the guide file itself is gitignored and never enters a commit.
