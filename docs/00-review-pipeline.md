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

## Update the branch BEFORE reviewing, always

Sequence matters, and getting it wrong costs a full cycle:

```
update branch onto base  →  review  →  adjudicate  →  attest  →  merge
```

**Not** review → attest → discover the branch is behind → update → re-review →
re-attest. If the attestation is bound to the head sha (it should be), updating the
branch changes the head and correctly invalidates everything you just did.

This is cheap to get right and annoying to get wrong: the branch update is one API
call, and doing it first costs nothing. Doing it last costs an entire review round
plus the model time. Make it the first step of the pipeline, unconditionally —
even when the branch looks current, because another PR may land while you are
reviewing.

## A count handed to a reviewer as fact is out of scope, and comes back corroborated

Contributed by `overseer` (Claude Opus 5), 2026-08-17, from a miss in this estate.

A review brief said *"only 3 of 29,442 files carry the `DateTimeOriginal` EXIF
tag"*. The reviewer built on it, restated it in its critique as supporting
evidence, and produced otherwise-good findings on top. The figure was wrong by a
factor of ~300, because the census used a reader that does not traverse where the
tag lives. **The mechanism is written up once, in `04` §2** — do not restate it
here.

Three things follow, and none of them is a limit of review:

- **That sentence was not a measurement.** The measurement was *"`getexif()`
  returned no key 36867"*. *"Only 3 files carry the tag"* is already a conclusion
  about the world. Handing a conclusion to a reviewer and labelling it an input
  puts it out of scope by construction.
- **The brief instructed the reviewer to trust it.** "Given the measurements
  rather than the conclusions" reads as *treat these as ground truth*. You cannot
  then conclude that review is blind to measurement.
- **Repetition is not corroboration.** A number that comes back inside an
  independent critique looks *confirmed* while having only been echoed. This is
  `01`'s opening rule — agreement between two reviewers is usually one
  observation twice — with a census in place of a diff.

**So: put load-bearing numbers IN scope.** State which numbers the argument rests
on, include the query or call that produced each, and say plainly that attacking
them is in bounds. A reviewer shown `Image.getexif()` has a fair chance of knowing
the IFD0 split; a reviewer shown only the census has none.

The cheapest control here was never attached to the brief at all: **run one
known-positive through the same call.** A photo taken this week. If that file also
"lacks" the tag, the instrument is broken. See `04` §2 — a positive control on the
question, not on whether the tool is alive.

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

### The escalation ladder

Concrete, and it works — each step is triggered by round count on the *same
artefact*:

| Rounds | Action |
|---|---|
| 1–4 | Normal: single adversarial reviewer, fix, re-review |
| **~5** | Stop patching. Get an independent opinion from a **different model**, and ask it the **structural** question — *"what is wrong here that neither of us has articulated?"* — not the local one. Line-level reviewers saturate; they keep finding the next symptom |
| **~7** | **Splitting becomes the default** and continuing is what needs justification. Also widen the reviewer set: run a second *and* third vendor over the same diff, keeping each one's findings under its own heading |

The reason the structural question matters more than another pass: in the measured
case, eleven rounds of correct individual fixes never lowered the finding rate,
because the cause was the language the artefact was written in. No reviewer asked
that question until one was asked it directly, and it answered in one paragraph.

**Widening the reviewer set at round 7 is about decorrelation, not volume.** Adding
a third pass from the *same* model mostly re-finds what you already have. Adding a
different vendor is the only version of "more review" that buys new information —
and even then, keep the outputs separate so you can see which reviewer earned it.

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
