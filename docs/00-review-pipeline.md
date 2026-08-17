# The review pipeline

How a change gets from an agent to `main`, and the two design decisions that
determine whether the pipeline costs minutes or hours.

---

## The pipeline

```
branch  →  build  →  local adversarial AI review
                  →  ADJUDICATE every finding in writing
                  →  post the adjudication (not the raw model output)
                  →  gate verifies an adjudication exists for this exact commit
                  →  merge
```

Three properties matter more than the specific tools:

**1. Review runs locally, before the PR is mergeable — not in CI.**
Moving it out of CI cut ~25% of an Actions bill (1,983 jobs/week for 766 minutes
of actual work; the rest was per-job minute rounding). CI's job is to *verify a
review happened*, not to perform it.

**2. Raw model output never reaches the pull request.**
What gets posted is an adjudication: every finding accepted or rejected, in
writing, with the reason. This is the load-bearing step. A model's findings are
input, not instructions — roughly 20% of confident, high-severity findings are
wrong, and the only thing standing between a false positive and a wasted hour is
someone writing down why it was rejected.

**3. The review runs in a throwaway worktree at the PR head.**
A reviewer must never read a checkout an agent is still mutating. Otherwise it
reviews a state that never existed.

## Adjudication is the deliverable

An adjudication that says "fixed" is worth little. One that is useful looks like:

- the finding, restated in one line;
- **accepted or rejected**, and why;
- for accepted: the fix, and the *executed* evidence it works — the command, the
  before/after, the exit codes;
- for rejected: the evidence that refutes it, checked against the governing
  source.

Rejections matter more than acceptances. They are what stops the same false
finding costing time in round 4, and feeding them back into later rounds measurably
reduces re-litigation.

## The gate, and its one expensive mistake

The gate should verify that an adjudicated review exists **for the exact commit
being merged**. Bind it to the head sha: without that, a review of an early commit
keeps vouching for everything pushed after it.

The temptation is to also bind the **base** sha — after all, `base...head` is the
diff that was reviewed, and if the base moves, that diff changed.

**Binding to the base branch's live tip is a throughput disaster, and it is worth
understanding why before copying this design.**

Every merge to `main` moves the tip. So every merge **invalidates the attestation
on every other open pull request**, each of which must then be re-reviewed and
re-attested. With *n* open PRs that is O(n²) review work, and it forces strict
serialisation: no two PRs can be prepared in parallel, because landing either one
invalidates the other.

Measured on a 12-PR queue this was the single largest cost in the pipeline —
larger than model time, larger than CI.

### What to do instead

Bind the attestation to the head sha **and to the merge-base**, not to the base
branch's moving tip. The merge-base is the commit the reviewed diff was actually
computed against; it does not change when unrelated work lands.

If you want strictness beyond that, invalidate on base movement **only when the
base advance touched files the PR's diff also touches**. That is the real
condition under which "the diff changed" — and it is cheap to compute:

```bash
git diff --name-only "$OLD_BASE".."$NEW_BASE" > /tmp/base-changed
git diff --name-only "$MERGE_BASE"...HEAD      > /tmp/pr-changed
comm -12 <(sort /tmp/base-changed) <(sort /tmp/pr-changed)   # non-empty ⇒ re-review
```

Anything else re-reviews a diff that did not change.

## Rounds, and when to stop

Track findings per round. The number falling is the signal you want.

**If the count stops falling, that is a design signal, not a code-quality signal.**
Measured on one file: findings went `5, 3, 7, 4, 2, 5, 5, 3, 3, 4, 7` across
eleven rounds, and a large share of later findings were defects introduced by the
previous round's own fixes.

At that point another round is the wrong move. The artefact is too large or too
wrongly-shaped to re-verify, and the correct responses are:

- **split the PR** — remove everything not required for the guarantee it makes;
- **question the shape** — in the case above, a shell script had accumulated YAML
  parsing, three independent date validators, locale handling, staging-state
  management and a compliance gate, in a language where every guard fails open by
  default. The individual fixes were all correct. The rate did not fall because
  the language was the problem.

**A concrete escalation rule that works:** past ~5 rounds on the same artefact,
stop patching and get an independent opinion from a *different* model — and ask it
the structural question ("what is wrong here that neither of us has articulated?"),
not the local one. Past ~7, treat splitting as the default and continuing as the
thing needing justification.

## Cost notes

- **Reviewing locally is cheaper than reviewing in CI**, by a lot, and the
  feedback loop is seconds instead of minutes.
- **A false positive costs more than a missed nit.** Instruct reviewers to say
  "I am unsure" rather than pad the list, and to state a concrete failure scenario
  — inputs/state → wrong output — for every correctness claim. A finding with no
  failure scenario is not a finding.
- **Scope the review to the diff.** Pre-existing defects are real but belong in a
  separate section and become issues. An author patching pre-existing defects
  inside a fix PR grows the diff, which grows the surface, which produces more
  findings. Separating them is what lets a PR converge at all.
