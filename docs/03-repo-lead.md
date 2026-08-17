# The repo lead protocol

One accountable agent per repository. This exists because concurrent agents in one
working tree destroy each other's work, quietly.

---

## The failure it prevents

Two real incidents, both from agents that were each behaving correctly:

- One agent ran `git add -A` in a shared worktree and swept another agent's
  in-progress state files into an unrelated commit.
- One agent ran a "read-only diagnostic" that included `git checkout <ref>`.
  Read-only in intent; it changed `HEAD` and silently deleted the other agent's
  untracked files.

Neither agent did anything unreasonable in isolation. The coordination was missing.

## The rule

**Exactly one agent holds the lead on a given path at a time.** The lead is the
only agent that may mutate the working tree, stage, commit, push, or merge within
that path.

### Scope the lead to a PATH, not to a repository

An earlier version of this document said "one agent per repository". That is the
right rule for a repo owned by one domain and the wrong one as soon as a repo is
shared — which happened to this repository within a day of it existing.

**This repo is the worked example.** Three domains with different constraint
levels, three agents, one repo. A single lead would have meant either one agent
writing content for domains it does not operate in, or two agents waiting.

So `.agent-lead.yml` maps **paths** to leads:

```yaml
leads:
  - path: /            → co-owned    (changes proposed by PR to all leads)
  - path: /RIA         → pw-cli
  - path: /Business    → ryg-lenovo
  - path: /Personal    → overseer
```

Resolution: **find the longest path prefix matching what you are about to change.**
That path's lead owns it.

Two consequences worth stating, because they are what make co-leadership work
rather than just relabelling the problem:

- **A shared root needs a different rule from an owned subtree.** Root here is
  co-owned, and changes there go through a PR the other leads can see. Landing
  directly in a shared path is the thing the whole protocol exists to prevent, and
  being *a* lead does not exempt you.
- **Authority does not transfer with the idea.** Another lead can propose a
  practice for your path; they cannot approve it for your path. Where a path
  carries obligations the others do not operate under, only that lead can judge
  whether a practice clears the bar.

This also answers the obvious objection to the single-lead design: it does not
scale past one domain per repository, and most interesting repositories do not stay
that way.

Other agents may:

- read the repo,
- run reviews **in a throwaway worktree** at a specific ref,
- open issues, comment on pull requests,
- work in a *separate* worktree rooted at a different path.

Batch work spanning repositories is fine — the constraint is per-repo, not
per-organisation. One agent per repo, several repos in parallel.

## What this does NOT cover — read this before trusting it

Added after two operators independently attacked the framing. Both were right, and
the omission mattered more than the protocol.

**This governs working trees. A growing share of shared mutable state is not one.**

Not covered, and not made safe by anything below:

- **Shared databases.** Two agents writing the same Postgres over MCP. One
  archived a record while the other was reading it; a deployed change altered the
  second agent's search results *mid-conversation*.
- **Shared cloud storage** both agents hold credentials for.
- **systemd units, container state, a branch-protected remote.**

`git worktree list` detects none of that.

**A second class, which single-writer would also not have prevented** — these are
failures of *state nobody can see*, not of concurrent writes:

- **A commit on no remote at all** — a single copy, on one disk, on a host that has
  lost power before. `git status` reports clean, because it cannot see
  committed-but-unpushed. Found only by sweeping:
  ```bash
  git rev-list --count <branch> --not --remotes    # >0 ⇒ exists nowhere else
  ```
- **Merged but not deployed.** Two PRs merged at 15:05Z and 15:17Z while the
  running container had been built at 14:23Z. The filter everyone believed was live
  had never executed. Compare the running artifact against the merge you believe is
  live.
- **A stale build artifact outliving its source.** A gitignored `dist/` still
  exported a function deleted from `src/`, so a test importing the package read
  `dist` and passed against code that was gone.

So: **the class this protocol prevents is two agents writing the same tree.** For
shared services the right answer is probably attribution and reconciliation rather
than exclusion — but do not read this document as "coordination is solved". Saying
otherwise would make it exactly the thing this repo is about: a check that cannot
fail, reporting success.

## Declaring the lead

Keep it in-band and cheap. A file at the repo root, committed:

```yaml
# .agent-lead.yml
lead: pw-cli                    # the agent/session holding the lead
host: nick-pw                   # where it runs
since: 2026-08-17T13:00Z
scope: "landing the open PR queue"
contact: "GitHub issues, or the session's remote-control channel"
```

Any agent picking up work reads this first. If `lead` is not you and is not stale,
you do not mutate — you open an issue or message the lead.

Handing over is an edit to that file and nothing else, which keeps the record in
git where the rest of the coordination state already lives.

### It is documentation, not a lock — and the distinction is load-bearing

Claiming the lead means committing and pushing a file **in the repo whose
mutations it governs**. Two simultaneous claims produce a merge conflict in the
lock file itself. Worse in practice: on a branch-protected `main`, committing it
requires a branch, a PR and a merge — so claiming the lead becomes a multi-minute
operation, slowest exactly when it is most needed.

**So call it what it is.** Commit it once, up front, change it rarely, and treat it
as a *declaration of intent* that a human or agent reads — not as a mutex. A thing
called a lock gets trusted like one, and this cannot bear that weight.

### Expiry must require a positive act, not elapsed time

An earlier draft of this document said: if `since` is older than your session
length, the lead is stale and may be claimed.

**That is the alerting `for:` trap, and this estate has already paid for it.** A
`CoordExporterDown` alert with `for: 1800s` sat pending through a 27-minute outage
and notified nobody. A duration guard tuned to avoid false positives silently stops
covering the real event.

Same shape here. Too long and the lock outlives a dead agent. Too short and a
legitimately slow lead is pre-empted **mid-write** — worse than no lock, because
now two agents both believe they hold it.

If you want expiry, make it a **heartbeat the lead rewrites**, so absence of
evidence is never read as evidence of absence. If you are not willing to maintain a
heartbeat, prefer an explicit handover and accept that a dead agent's claim needs a
human to clear.

### Fetch before you read it

The pre-flight is `git fetch && cat .agent-lead.yml`, **not** `cat`. A stale local
checkout shows a stale lead, confidently.

This matters more than it looks on multi-host estates: some dispatch mechanisms
deliver only at the recipient's next session start, so a lead claim made while
another agent is running may not reach it for hours. The committed file helps only
if something makes the other agent fetch and read it — and nothing does
automatically. A live message channel is the only thing that arrives mid-session,
and a committed file is not one.

## Pre-flight, before any mutation

Cheap, mechanical, and catches the real cases:

```bash
git worktree list                 # is anyone else rooted here?
git status --porcelain            # uncommitted work that is not yours?
cat .agent-lead.yml 2>/dev/null   # who holds the lead?
```

If the tree is dirty and the changes are not yours, **stop**. Do not stash, do not
clean, do not checkout. Those are the three commands that destroy other agents'
work.

## Rules that survive contact

- **Stage by explicit path. Never `git add -A` or `git add .`** in a tree another
  agent could touch. This one rule prevents the most damaging incident class, and
  it is mechanically enforceable — a pre-tool guard rejecting the pattern works.

  **Watch the form that feels safe and is not: `git add -A <paths>`.** Appending
  paths reads like scoping, and `-A` still governs. A guard caught exactly this,
  from an agent that had just written the rule down.
- **A "read-only" operation that runs `git checkout`, `git pull`, `git switch`,
  `git stash`, `git clean` or `git reset` is a mutation.** To inspect history
  without touching the tree:
  ```bash
  git show <ref>:<path>            # one file at a ref
  git diff <ref1>..<ref2> -- <path>
  git archive <ref> | tar -x -C /tmp/inspect/   # whole tree, elsewhere
  ```
- **Reviewers get a throwaway worktree**, never the live checkout — an agent may be
  mid-edit, and a review of a state that never existed is worse than no review.
- **Return the repo to its default branch, clean, at session close.** A repo left
  on a stale branch with untracked files trips the next session's pre-flight and
  costs the first ten minutes of every subsequent session.
- **Editing a script while it is running is a mutation with teeth.** `bash` reads
  scripts incrementally; editing one mid-execution can corrupt the running
  instance. Wait for it to finish.

## Escalation

If two agents genuinely need the same repo at once, the answer is a second
worktree at a different path, not turn-taking within one tree:

```bash
git worktree add ../repo-agent-b <branch>
```

Separate paths, separate indexes, no interference. Merge through pull requests as
usual.

## Why not a lock server

Because the coordination problem is small and the failure mode is human-legible.
A committed file that any agent can read, and that shows up in `git log` when it
changes, is enough — and it degrades gracefully when an agent dies mid-session,
which a lock server does not.
