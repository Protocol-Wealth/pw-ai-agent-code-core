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
