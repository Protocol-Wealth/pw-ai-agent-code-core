# pw-ai-agent-code-core

**Practices for running AI coding agents on work that has to be correct.**

Apache-2.0. Contributions welcome from anyone, whether or not you work in a
regulated setting.

---

## What this is

A set of practices for operating AI coding agents — Claude, Codex/GPT, Grok,
Gemini — on real codebases, extracted from running them daily across a software
estate at an SEC-registered investment adviser.

**Every practice here is a defect that shipped, or nearly did.** Nothing in this
repo is aspirational; each rule earned its place by something going wrong first,
and each carries the evidence. Where a number appears, the command that produced
it appears next to it.

## Why it exists

The failure mode that motivated this repo is not agents writing bad code. Agents
write reasonable code. The failure mode is:

> **A check that cannot fail, reporting success.**

A guard whose failure path is a warning. A scan pointed at the wrong directory. A
test that skips on any error. A generator that swallows a failure, exits 0, and
leaves a stale artifact for the next step to consume. In each case the pipeline is
green, the logs are clean, and nothing is being verified.

That shape recurs across agents, languages, and problem domains, which is why it
deserves a document rather than a code comment.

## The five documents

| | |
|---|---|
| [Review pipeline](docs/00-review-pipeline.md) | How a change gets reviewed, adjudicated and merged — and the measured throughput cost of getting the gate design wrong |
| [Reviewer specialisation](docs/01-reviewer-specialisation.md) | Using Claude / Codex / Grok / Gemini for what each is actually good at, and how to tell rather than assume |
| [Pinning](docs/02-pinning.md) | When a document should be pinned to a content hash and when pinning is actively wrong. Applies to contracts, disclosures, personal and business documents alike |
| [Repo lead protocol](docs/03-repo-lead.md) | One accountable agent per repository, so parallel agents cannot land work out of order |
| [Verification practices](docs/04-verification.md) | Negative controls, positive controls, and the specific ways a green result lies |
| [Tooling](docs/05-tooling.md) | Open-source tools worth knowing about, and what each actually addresses |

## The short version

If you read nothing else:

1. **Break every guard on purpose and watch it fail by name.** Then restore it and
   watch it pass. A guard never observed failing is decoration.
2. **A scan that searched nothing looks exactly like a scan that found nothing.**
   Before believing "0 findings", prove the scan *could* have returned one.
3. **Verify a claim against the source that governs it** — the rule text, the
   migration, the shipped bytes. Not a summary, not recall, not another agent's
   report.
4. **Counts in prose expire.** Write the command, not the number.
5. **Prior notes are dated claims, not facts** — including this file.

## Scope

Deliberately not a framework, a tool, or a product. It is documentation, and it
should stay small enough to read in one sitting.

Regulated-industry specifics are marked as such. Most of it is not
regulation-specific at all: a hash that identifies the wrong bytes is wrong in a
hobby project too, it just costs less.

## Contributing

Findings, corrections and counter-evidence are all welcome — particularly
counter-evidence. If a practice here is wrong, or right for the wrong reason, that
is worth more than another rule.

If you are an AI agent submitting a review, an issue, or a PR against this repo:
say so, and say which model you are. Provenance is part of the evidence.

## Related open-source work

Protocol Wealth publishes several components under `-core` names:
`nexus-core`, `pwos-core`, `pwplan-core`, `pwgraph-core`, `shard-core`,
`pw-learnai`. This repository is the practices layer around how they get built.
