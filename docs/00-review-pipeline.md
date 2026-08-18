# The review pipeline

How a change gets from an agent to `main`, and the two design decisions that
determine whether the pipeline costs minutes or hours.

---

## The pipeline

```
branch  →  build  →  local checks (lint/typecheck/test) on a pre-push hook
                  →  adversarial AI review, WHEN IT IS WARRANTED
                  →  ADJUDICATE every finding in writing
                  →  post the adjudication (not the raw model output)
                  →  merge
```

> **This pipeline used to end in a gate: a required CI check verifying an
> adjudication existed for the exact commit. We built that, ran it, measured it,
> and deleted it on 2026-08-18. The section
> [The gate, and why we deleted it](#the-gate-and-why-we-deleted-it) is now the
> most useful part of this document, because it is the part where the evidence
> contradicted us.**

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

## The gate, and why we deleted it

We ran a required CI check that verified an adjudicated review existed for a
PR's exact head before it could merge. It is gone. The measurement is the useful
part, so here it is rather than the design:

| | |
|---|---|
| stored review runs | **366** |
| rounds on a single PR | **19** (also 16, and 15) |
| model calls per round | 2 |
| rounds that changed an outcome | a small minority |

On the one pull request that ran five rounds to exhaustion, **the actual blocker
was found afterwards by a single structural pass from a different model.** The
five line-level rounds never found it; each round's findings landed mostly in the
previous round's fixes.

**Nothing required the gate.** Our governance register has no requirement for
code review, peer review, change control, segregation of duties or an SDLC. It
was a firm practice guarding itself — and by the end it had grown a 182-line
script whose only job was asserting the *shape* of the gate, run by its own
workflow. Machinery guarding machinery.

### The failure worth copying is what we did BEFORE deleting it

In a single day, one agent built **three separate things to make that ceremony
cheaper**: a settle step so the gate stopped needing a manual rerun after every
attestation; a documentation carry-forward so docs-only commits would not
re-trigger a full review; and a moving tag with its own currency check so a gate
change stopped costing five pull requests across five repositories.

Every one of those was locally correct. All three were deleted the same week,
along with the thing they served.

The doctrine that produced it is in this very repository: *prove every guard by
negative control; a check that cannot fail is decoration.* Applied recursively,
with no stopping rule, it **manufactures** guards for guards. Nothing asked the
zeroth question:

> **Is the thing being guarded required?**

The answer was already filed, in a register of things explicitly NOT required.
The rule existed in an agent's memory notes; it did not exist in any document
the agent loaded.

### The rule that replaces it

Put this at **position 0** of any planning checklist, before the questions about
scope and data model:

> **If this work builds, hardens, or cheapens a control, gate or ceremony: cite
> the requirement that mandates the control — verbatim, from the governing
> source — before writing anything. No citation means the candidate action is
> DELETION, raised with the owner immediately, not after you have made it
> cheaper. Optimising an unrequired control is the same defect as guarding a
> constraint you imposed on yourself.**

And append this to *a check that cannot fail is decoration*:

> A check verifying a control nobody requires is **also** decoration, however
> well it fails.

### What review looks like without a gate

Reviewing is still worth doing. It caught a P1 inside a fix for another P1 on
the same day we deleted the gate. **Stamping that it happened was the waste.**

- substantive code changes: review **once**, not per push
- **structurally first** when a control, compliance surface or schema is involved
- **not at all** on documentation, pins, chores or dependency bumps
- **stop at two rounds** — see [Rounds, and when to stop](#rounds-and-when-to-stop)

Move the mechanical checks to a **pre-push hook** instead. Ours runs lint,
typecheck and unit tests (or `terraform fmt` plus the verifier self-tests). The
economics are not subtle: CI and deploy were **85% of our Actions wall-clock**,
and GitHub bills every job **rounded up to a whole minute** — so a red push is
never one wasted minute, it is a full job fan-out, and the fix costs another.

### The one review we kept in CI

Dependency-bot pull requests, because they **auto-merge on minor and patch with
nobody at a keyboard**. That is the line worth drawing:

> **CI reviews what no human initiated. Local reviews what a human did.**

A local-only review requires a human at a terminal. Anything that merges itself
does not have one.

## A count handed to a reviewer as fact is out of scope, and comes back corroborated

Contributed by `personal-lead` (Claude Opus 5), 2026-08-17, from a miss in that estate.

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
| 1–2 | Normal: single adversarial reviewer, fix, re-review |
| **2** | **If the count has not fallen, STOP.** Get an independent opinion from a **different model**, and ask it the **structural** question — *"what is wrong here that neither of us has articulated?"* — not the local one. Line-level reviewers saturate; they keep finding the next symptom |
| **3+** | **Splitting is the default** and continuing is what needs justification |

**This ladder used to say 1–4 normal, ~5 structural, ~7 split.** It was revised
down on 2026-08-18 against the measurement above: rounds 3–5 on the PR that ran
to exhaustion produced findings *in the previous round's fixes*, not in the
original work, and the structural pass that actually solved it would have been
correct at round 2. The old numbers were an estimate; these are what the counting
showed.

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
