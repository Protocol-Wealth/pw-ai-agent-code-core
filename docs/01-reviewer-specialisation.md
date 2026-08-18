# Reviewer specialisation

Using more than one model well, and — more importantly — knowing when you are
entitled to say one is better than another.

---

## Start here: agreement between two reviewers is usually not corroboration

Two models reading the same truncated diff is **one observation counted twice**.
Their errors are correlated: same context, same omissions, same misreading.

Measured instance: a three-model CI panel produced three *convergent* findings, of
which **two were false**. Convergence felt like signal and was not.

So:

- **Never merge reviewers' findings into one list.** Keep each reviewer's output
  under its own heading. Where they agree, that agreement must remain *visible as
  agreement* so it can be weighed. Flattening destroys the only thing a second
  reviewer was added to provide.
- **Gate on the adjudication, not on the vote.** Two models agreeing is not a
  reason to accept; one model with a reproducible failure scenario is.
- **Diversity beats redundancy.** If you run two reviewers, give them *different
  lenses* (correctness, security, "does this reproduce", regulatory citation) or
  different vendors. Two identical passes mostly buy you the same finding twice.

## What we can and cannot claim about specific models

Being honest about the evidence base, because this is exactly where unfounded
rankings get repeated until they are treated as fact.

**What is measured, from daily use:**

- **Codex / GPT (`codex exec`, read-only sandbox)** — strong as an adversarial
  second reader of a diff it did not write. In one file it found ~15 real defects
  including a command injection in a cleanup trap, an arbitrary-file-deletion via
  an inherited environment variable, and an incorrect regulatory citation. It also
  produced at least 3 confident false positives in the same period, including one
  that inverted a documented, previously-verified result. Both halves are the
  characterisation.
- **Claude** — used as the implementing agent and adjudicator: multi-file
  orchestration, holding cross-repo state, and writing the adjudication. Reviewing
  its own work is the weakest use; it shares its own blind spots.
- **A second opinion from a different model on a *decision*** (stop or continue,
  is this design wrong) has been high-value where line-level review had saturated.
  In one case it correctly identified that nine rounds of findings on one file were
  symptoms of the language choice, and caught that filing an issue while leaving a
  known-false claim in an operator-facing document was "the wrong half of the fix".

**What is NOT measured, and should not be asserted:**

- Whether Grok is better than Codex, or Gemini than either, **at anything**.
  See the two-lane data below: it is a *disjointness* result, not a ranking, and
  it does not support one.

## Measured: two lanes over identical diffs, seven rounds

Contributed by another operator running `codex` and `grok` over the **same diff**
from a detached worktree, with every finding adjudicated against the code by hand
rather than counted:

| round | subject | codex | grok | adjudication |
|---|---|---|---|---|
| 1 | supersession | 2 | 0 *(clean verdict)* | both codex findings real |
| 2 | supersession | 1 | 0 *(clean verdict)* | codex **refuted** — misattributed a line to the wrong function |
| 1 | pricing | 0 *(clean verdict)* | 2 | both grok findings real |
| 2 | pricing | 1 | 0 *(clean verdict)* | codex real — a *second* copy of a rate table the author had missed |
| 3 | pricing | 0 | 0 | converged |
| 1 | contributor | 4 | 1 | all 5 real; both lanes independently found the same read-side defect |
| 2 | contributor | 3 | 2 | all 5 real |
| 3 | contributor | 2 | 0 *(clean verdict)* | both real |

**Totals: codex 13 findings / 12 real / 1 refuted. Grok 5 findings / 5 real / 0 refuted.**

### Read this correctly — it is not a ranking

It is tempting to read "12 real vs 5 real" as codex winning. That is the wrong
conclusion and the data says so:

- **On the pricing branch, grok found two real defects while codex declared the
  diff clean.** On supersession round 1, the reverse.
- Codex emits more findings and issues a clean verdict less readily; grok emits
  fewer and refused nothing. Those are different operating points, not different
  skill levels.

**The finding is disjointness.** Across seven rounds the two lanes' findings
overlapped **exactly once**. A separate PR in the same estate produced fully
disjoint findings. On this evidence, **single-lane review would have shipped
roughly half of these defects** — and which half depends on which lane you picked,
not on which is better.

That is the actual argument for a second reviewer, and it is much stronger than
"two opinions are better than one".

### Verdict lines are not summaries

Both lanes produced misleading top-line verdicts. Codex opened one review with
*"internally consistent … without introducing a behavioral regression"* — and grok
then found two real defects in that same diff.

**Never gate on a reviewer's summary sentence.** Read the findings. A confident
clean verdict is the single most expensive thing a reviewer can be wrong about,
because it is the one nobody checks.

### A second, larger sample — and the reliability axis the first one missed

A later session on the same harness ran **27 review runs** across three repositories (an
application, a database-backed service, and shell operations code), same prompt, same
diffs. The denominators differ per lane and that difference is the finding, so state the
sample precisely rather than as one number:

| | |
|---|---|
| review runs total | 27 |
| of those, TWO-LANE attempts | 21 (six were deliberately single-lane during iteration) |
| runs where **both** lanes completed | **14** — the paired sample |
| lane A | completed 25 of 25 attempts |
| lane B | completed 14 of 19 attempts (4 clock timeouts, 1 turn-budget truncation) |

**Disjointness is measured over the 14 paired runs**, not over all 27: a round where one
lane never answered cannot show whether the two agree. Overlap across those was 3. Each
lane was again the sole finder of real defects. The clearest single case: on one change
lane B raised the P1 that invalidated the author's *entire approach* — a fix that only
worked when a write happened to land inside a narrow sampling window — while lane A found
three separate fail-open paths in the same diff and missed the structural flaw. Neither
found the other's.

**Completion is a separate axis from finding rate, and it is invisible in a findings
table.** Lane B failed to finish roughly a quarter of its attempts, and on one particular
file it failed three times out of four while completing 94–110 KB diffs elsewhere in the
same session. Wall-clock tracked how much the agent chose to investigate, not diff size.

- **A timed-out lane is not a passed lane**, and a harness that reports the two the same
  way turns a one-lane review into a two-lane claim. Report per-lane outcomes.
- Do not tune the timeout down without the distribution — and do not tune it *up* without
  one either. Here the successful runs clustered just under the ceiling, which the
  operator first read as "the limit is correctly placed". Re-running one failure at three
  times the ceiling finished in **615 seconds**, fifteen past the old limit, with a full
  verdict and two real findings. Successes clustering just under a limit is equally the
  signature of a limit severing runs mid-answer; only re-running *past* the ceiling
  distinguishes the two.

**Scope:** one operator, one harness, two model CLIs, a single session. Enough for
disjointness and for the reliability asymmetry; **not** enough to rank the models, for
the same reasons stated below.

### What a proper comparison would still need

The table above is one operator, one harness, three branches. It is enough to
establish disjointness; it is not enough to rank. That needs the replay method
below, and one refinement contributed alongside the data: **score unique finds by
CATEGORY, not only by count.** The asymmetry lives in the categories — one
reviewer's unique finds were scope errors in evidence, a class self-review is
structurally bad at because the reviewer shares the author's framing of what the
population is.

## How to actually find out, rather than guess

Ranking reviewers is an empirical question with a cheap answer. Run it:

1. **Fix the corpus.** Take 20–30 *already-adjudicated* findings — real diffs where
   you know the ground truth because you investigated each one.
2. **Replay each reviewer over the same diffs**, same prompt, no shared context.
3. **Score what matters**, per reviewer:
   - **true positives** — real defects found
   - **false positives** — confident findings that were wrong (weight these
     heavily; they cost more than misses)
   - **unique finds** — defects *only* that reviewer found. This is the number that
     justifies paying for a second reviewer at all
   - **cost and latency** per review
4. **Publish the table with its date.** Model behaviour changes; a ranking is a
   dated claim like any other.

Until that exists, the defensible position is: run a second reviewer from a
*different vendor* because decorrelated blind spots are worth something, and keep
its findings separate so you can see which reviewer earns its cost.

## Assigning work by capability

Even without a ranking, some assignment is defensible on structure alone:

| Task | Assign to | Why |
|---|---|---|
| Implementation, multi-file orchestration | The agent holding the context | Cross-file coherence needs the whole picture |
| Adversarial review of that implementation | **A different vendor** | Decorrelated blind spots — the only reason a second opinion is worth anything |
| Adjudicating findings | The implementing agent | It can check each claim against the code and is accountable for the result |
| Stop-or-continue / "is this design wrong" | A different model, asked the **structural** question | Line-level reviewers saturate; a fresh reader sees the shape |
| Regulatory / citation checks | Whoever will actually open the rule text | Model choice matters less than the discipline of reading the source |

## Rules that hold regardless of model

- **A reviewer that returns nothing must be distinguishable from a reviewer that
  did not run.** If a review produces no output, that is INCONCLUSIVE, not clean.
  Require an explicit "no findings" statement, and if the transport failed, say so
  and keep the error — never discard stderr.
- **Never let a reviewer write.** Read-only sandbox, denied write tools. "Findings
  only" should be structural, not a polite request in a prompt.
- **Feed prior adjudications forward.** Without them, a finding rejected with
  evidence in round 2 returns in rounds 3 and 4 wearing the same confidence.
  Measured at ~15% of all findings across eight rounds.
- **Give reviewers the same brief.** Severity, file:line, one-sentence claim, a
  concrete failure scenario, and a confidence. No formatting or naming preferences.
