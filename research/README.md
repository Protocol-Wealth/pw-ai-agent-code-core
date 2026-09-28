# research/ — sources behind the practices, and what we have not read

Co-owned, domain-neutral. This directory exists so a practice can be traced to
something other than the confidence of whoever wrote it down.

## The rule for this directory

**Say whether you read it.** The same status vocabulary as `tools/`: **READ**
(opened the primary source), **CITED** (taken from a secondary source that
referenced it), **UNREAD** (surfaced by a search; title and abstract only).

**CITED and UNREAD entries are still worth listing** — the point is to make the
gap visible rather than to imply a literature review nobody did.

> A citation is not evidence for the claim it is attached to. Attaching a
> plausible-sounding source to a decision makes the decision *harder to revisit*
> while adding nothing, and a source that argues the opposite is worse than none.

---

## Model release watch — 2026-09-28

| source | status | what it supports |
|---|---|---|
| [Anthropic, *Introducing Claude Sonnet 5.5*](https://www.anthropic.com/claude-sonnet-5-5) | **READ** official announcement | Sonnet 5.5 is available under API identifier `claude-sonnet-5-5`. Anthropic reports 30%+ faster output than Sonnet 5 and up to 30% lower cost per task in its tests, at the same $2/M input and $10/M output list prices. The announcement also says zero data retention is available and flags a `between_tools` migration for applications that ran Sonnet with thinking off. These are vendor claims and availability statements, not results reproduced in this estate or proof of this firm's credential settings. |

**Use for model-selection work:** evaluate the new model on the same adjudicated
coding, review, and document tasks as the current pin; record task quality,
latency, token use, and cost per completed task. Verify the exact API behavior
and the account's data-retention terms before a pin change. The announcement
alone does not change a production model registry or review lane.

## Agent memory and consolidation

| source | status | what it supports |
|---|---|---|
| **`always-on-memory-agent`** (GoogleCloudPlatform/generative-ai) | **CITED** | Three design choices we adopted in argument: SQLite with **no vector DB** at decision-store scale (reading everything and synthesising beats semantic search when the corpus is ~17 documents); a **consolidator on a timer** that does not depend on any session choosing to write; and **importance scoring at ingest** to capture broadly without building a landfill. *Not independently reviewed.* |
| **DeepLearning.AI agent-memory course** | **CITED** | The lifecycle vocabulary in use throughout: **episodic** (transcripts) → **semantic** (consolidated facts) → **procedural**. Useful as shared language; not evidence for any mechanism. |

**The open objection, recorded because it is unresolved:** a consolidator reading
transcripts inherits an unreliable narrator. Transcripts record *both sides of
every argument*, so a consolidator can promote "X is under reconsideration" into
the semantic store **as fact**, with a store entry's authority — poisoning the
record it exists to protect. A cheaper component addresses the same diagnosis:
compare what a session **did** against what it **wrote**, and alarm on the
mismatch. That measures absence; a consolidator manufactures presence.

### Memory and context references reviewed 2026-09-28

The **READ** status below means the project's README or the linked primary
documentation was opened, not that its code was audited, installed, or run.
These sources sharpen the open objection above; none establishes a new practice
in `docs/` without a concrete failure and a check that can fail.

| source | status | what it supports or challenges |
|---|---|---|
| [Mem0](https://github.com/mem0ai/mem0) | **READ** README | Multi-scope, additive facts and combined retrieval are candidate patterns. Its 2026 headline benchmark scores are explicitly for its managed platform, not the open-source SDK. |
| [Hindsight](https://github.com/vectorize-io/hindsight) | **READ** README | Separate retain, recall, and reflect operations, plus multiple retrieval signals. Reflection needs a clear boundary between evidence and model interpretation. |
| [memU](https://github.com/NevaMind-AI/memU) | **READ** README | Session-to-skill distillation exposes the exact transcript-to-authoritative-instruction promotion risk described above. Its host adapters can patch agent instruction files. |
| [Cognee](https://github.com/topoteretes/cognee) | **READ** README | Explicit remember, recall, improve, and forget lifecycle; session knowledge can be promoted into longer-lived memory. |
| [Graphiti](https://github.com/getzep/graphiti) | **READ** README | Dated fact validity and episode provenance are useful checks against stale or unsourced claims. |
| [OpenViking](https://github.com/volcengine/OpenViking) | **READ** README | Scoped directory search and abstract/overview/detail loading can reduce unnecessary context. Main-project AGPL-3.0 licensing rules out copying code into permissively licensed projects. |
| [Letta](https://github.com/letta-ai/letta) and [Letta Code](https://github.com/letta-ai/letta-code) | **READ** READMEs | Letta points to Letta Code as current source; editable, versioned agent context is useful prior art and a reminder that self-edits need review. |
| [OpenMemory](https://github.com/mem0ai/openmemory) | **READ** README | Selective cross-harness session transfer is possible; moving a raw transcript changes its data boundary. Realtime autosync is listed as planned. |
| [Agent Memory Benchmark](https://github.com/vectorize-io/agent-memory-benchmark) | **READ** README | Retrieval-only cases can mark irrelevant memories as failures, rather than hiding them in answer quality. The benchmark authors also develop Hindsight, so compare independent baselines. |
| [XDA local-agent context report](https://www.xda-developers.com/stopped-my-local-llm-agent-from-running-out-of-context/) | **READ** article (secondary) | One hardware-specific account of context exhaustion and runner configuration. Verify settings against [LM Studio's load API](https://lmstudio.ai/docs/developer/rest/load) (**READ** primary) before turning numbers into advice. |

### Agent context and instruction efficiency reviewed 2026-09-28

**READ** means the linked primary README, PR, or full paper was opened. The XDA
article was read through a text mirror because its site blocked direct retrieval;
it is a personal workflow report, not a controlled measurement. None of these
sources has been run as a tool or reproduced on this repository.

| source | status | useful observation and limit |
|---|---|---|
| [Backpass](https://github.com/kunchenguid/backpass) | **READ** README | It proposes instruction edits from actual session transcripts, keeps analysis separate from application, and attaches session evidence to proposals. Its two-session evidence threshold helps filter one-off anecdotes; human review and behavior tests still decide whether a rule moves. Transcripts can contain sensitive data, so scope and redaction need review before using a hosted agent to analyze them. |
| [FirstMate instruction PR #5872](https://github.com/kunchenguid/firstmate/pull/5872) | **READ** PR | A reported roughly 48% reduction moved situational guidance from a large `AGENTS.md` into on-demand skills. A side-by-side behavior check exposed lost or late-triggered guidance, prompting follow-up edits. The transferable check is task success **and** skill trigger timing against representative old sessions, alongside token cost. Its result is a case study, not a target percentage for other repos. |
| [JAZ paper, *Harness as a Language*](https://arxiv.org/html/2609.26891v1) | **READ** full paper | Exposing the full interaction history as a programmatically searchable variable let an agent recover exact old details after context handoff; hooks supplied budget, validation, recursion, and trajectory controls. In the authors' StuLife far-recall subset, JAZ reached 69.9% pass rate versus Letta's 61.8% across three runs; AppWorld self-improvement used six runs. Those benchmark-specific results motivate a retrieval comparison, not replacing a deployed memory system without local evaluation. |
| [XDA token-saving workflow report](https://www.xda-developers.com/claude-code-token-saving-tricks-nobody-talks-about/) | **READ** article (secondary) | Scope known work to relevant paths; choose reasoning effort for task difficulty; keep stable repo rules in the project guidance file and one-off constraints in the task; reduce PDF noise before deeper analysis. For this estate the shared entry point is `AGENTS.md`. Measure whether a shorter prompt or PDF digest omits evidence before claiming savings. The article supplies no controlled comparison across repositories. |

**Candidate evaluation, not adopted policy:** take a small set of representative
tasks, including ones that need rare compliance or release rules. Record baseline
instruction tokens, whether each required rule was loaded at the right time,
task outcome, and cost. Move a rule to an on-demand skill only when those tasks
still pass and the skill trigger is observable. For long-history tasks, compare a
summary-only handoff with exact-source retrieval and check whether citations point
to the original transcript entry rather than the agent's recollection. Keep raw
transcripts out of public repositories and preserve a human review step before
instruction or durable-memory edits.

## Verification and review method

| source | status | what it supports |
|---|---|---|
| **Mutation testing on an extraction pipeline** (measured in-estate, 2026-08-17) | **READ** | Baseline 56.50%, 117 surviving mutants. 1,476 tests stayed green while a whole extraction path was disabled. Measure the baseline **before** setting a threshold, or the first CI run fails for a reason nobody chose and the gate gets deleted. |
| **python-pillow/Pillow #6641** | **CITED** | Documents that `getexif()` without further processing returns IFD0. Cited by a reviewer; **not opened by the author of this entry.** Listed precisely because it shows the bug in `docs/04` §2 was publicly known and findable. |

## Perceptual hashing and near-duplicate detection

| source | status | what it supports |
|---|---|---|
| *Comparative Evaluation of Perceptual Hashing and Deep Embedding Methods for Robust and Efficient Image Deduplication* — MDPI Electronics 15(7):1493 | **UNREAD** | Compares AHash/DHash/PHash/WHash against CNN embeddings. Surfaced by search; abstract only. Would settle which method to prefer, and nobody has read it. |

## Face recognition on children — an open, load-bearing gap

Relevant wherever a personal archive is indexed by person and the subjects are
minors. **All three UNREAD**; listed because a self-hosted photo tool was
selected partly on this and the underlying literature was never consulted.

- *Evaluating Deep Learning-Based Face Recognition for Infants and Toddlers: Impact of Age Across Developmental Stages* — arXiv 2601.01680
- *Mitigating Longitudinal Performance Degradation in Child Face Recognition Using Synthetic Data* — arXiv 2601.01689
- *Child Face Age-Progression via Deep Feature Aging* — arXiv 2003.08788

The practical finding that prompted the search **is** verified: PhotoPrism
documents that children's faces are not reliably recognised because its model was
trained on North-American public-licence images, which largely exclude children
(their issue #1587). The papers would say whether that is a PhotoPrism problem or
the state of the art.

## Prior art on agent conventions

| source | status | what it supports |
|---|---|---|
| **`deepseek-ai/deepseek-harness` `AGENTS.md`** | **READ** 2026-08-17 (150 lines, via `gh api`) | Conventions for AI agents contributing to a repo, from an org with weight behind it. Three rules transfer directly and are now in `docs/04` §4b and §4c, credited there: *report only commands run*; *match evidence to the surface, never default to the full suite*; and **give time-bound guidance an expiry condition in its own text** (their opening section is titled "Pre-release stance" and begins "Remove this section at the first tagged release"). |

---

## Contributing an entry

Give the source, the status, and **what practice it actually supports**. A source
that supports nothing we do is a reading list, not research. If a practice here
turns out to rest on an UNREAD source and someone then reads it, **update this
file first** — a superseded belief with a live citation is the most expensive
shape in the repo.
