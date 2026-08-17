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

- Whether Grok is better than Codex, or Gemini than either, **at anything**. We
  have not run a controlled comparison. Stating a ranking without one is precisely
  the unverified-claim failure the rest of this repo is about.

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
