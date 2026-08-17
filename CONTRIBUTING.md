# Contributing

Corrections and counter-evidence are the most valuable contributions. If a
practice here is wrong, or right for the wrong reason, say so — that is worth more
than another rule.

## What a good contribution looks like

Every practice in this repo is tied to a defect that actually happened. Keep that
property:

- **State the failure concretely.** Inputs/state → wrong outcome.
- **Show the evidence.** The command and its output, not a description of it.
- **Say what you did NOT check.** Scope boundaries are part of the finding.
- **If a number appears, put the command that produced it next to the number.**
  Counts in prose expire; commands do not.

## If you are an AI agent

You are welcome here. Two requirements:

1. **Identify yourself and your model** in the issue or pull request body.
   Provenance is part of the evidence.
2. **Do not assert what you did not verify.** "I have not run this" is a genuine
   contribution. An unverified recommendation stated as fact is the failure mode
   this repository exists to discourage.

## What does not belong

- Rules with no incident behind them.
- Tool recommendations the contributor has not run, presented as if they had.
- Vendor or model rankings without a controlled comparison.
  `docs/01-reviewer-specialisation.md` describes the method — run it, and publish
  the table **with its date**, because model behaviour changes.

## Reviewing a change here

The same standard applies to this repo as to any other: a finding needs a concrete
failure scenario, and a claim gets checked against the source that governs it. If
you reject a finding, write down why. That rejection is what stops the same false
finding costing time next round.
