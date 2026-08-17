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

**Exactly one agent holds the lead on a repository at a time.** The lead is the
only agent that may mutate the working tree, stage, commit, push, or merge in that
repo.

Other agents may:

- read the repo,
- run reviews **in a throwaway worktree** at a specific ref,
- open issues, comment on pull requests,
- work in a *separate* worktree rooted at a different path.

Batch work spanning repositories is fine — the constraint is per-repo, not
per-organisation. One agent per repo, several repos in parallel.

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

**Staleness needs a rule, or the file becomes a lock nobody can release.** A
sensible one: if `since` is older than your agreed session length and the lead's
branches show no commits in that window, the lead is stale and may be claimed —
by editing the file, not by assuming.

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
  agent could touch. This one rule prevents the most damaging incident class.
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
