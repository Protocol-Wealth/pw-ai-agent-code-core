# Tooling

Open-source tools worth knowing about when running AI coding agents, grouped by
the problem they actually address.

**This is a gap-check, not a shopping list.** A tool appearing here is not
evidence you need it. Several entries are listed with the caveat that we have not
run them — stated explicitly, because an unverified recommendation is the thing
this repo exists to discourage.

---

## Verified in daily use

| Tool | Licence | What it addresses |
|---|---|---|
| [`ripgrep`](https://github.com/BurntSushi/ripgrep) | MIT / Unlicense | Linear-time search. **Matters more than it sounds:** some `grep` implementations (notably `ugrep`) compile bounded repetition `.{0,N}` into an automaton whose memory is superexponential in N — a 27-byte input reached multi-GB at N=100 and took a host down. Prefer `rg`, or keep bounds ≤ 50. |
| [`qpdf`](https://github.com/qpdf/qpdf) | Apache-2.0 | Decompressing PDF object streams. Essential when checking whether a structure is *actually* present: grepping raw PDF bytes for `/OutputIntents` returns 0 even when it is there, because object streams are compressed. |
| [veraPDF](https://github.com/veraPDF/veraPDF-library) | GPLv3 / MPLv2 | The reference PDF/A validator. Note it **dispatches on file extension** — hand it `x.pdf.tmp` and it silently declines to parse, yielding no verdict rather than an error. |
| [`trivy`](https://github.com/aquasecurity/trivy) | Apache-2.0 | Filesystem and IaC scanning; runs locally, so no per-PR CI minutes. |
| [`gh`](https://github.com/cli/cli) | MIT | GitHub from scripts. **Pass issue and PR bodies with `--body-file`, never `--body "…"`** — unescaped backticks in a double-quoted shell string are command substitution and will silently delete code spans from your text. |
| [`gh act`](https://github.com/nektos/act) | MIT | Run GitHub Actions locally. Directly reduces the Actions bill, which for agent-heavy repos is often the dominant cost. Invoke as `gh act`. |
| [`jq`](https://github.com/jqlang/jq) | MIT | Structured output from APIs. Preferred over parsing text: reading both shas from **one** `pulls/N` call is atomic where three separate calls can disagree. |

## Worth evaluating — not yet run here

Listed because the problem is real and recurring, with the specific failure each
would address. **We have not evaluated these; treat as leads.**

| Tool | Problem it targets |
|---|---|
| [`nah`](https://github.com/ai-boost/awesome-harness-engineering) *(via the harness-engineering list)* | Deterministic permission guards mapped to an **intent taxonomy** rather than command-name allow/deny lists. The enforcement layer prompt-level scoping cannot provide — a subagent described as read-only reaching production is a prompt problem that only a deterministic guard fixes. |
| Credential brokers (e.g. Agent Vault) | Injecting credentials onto outbound requests instead of putting them in config, argv or environment — where `${VAR:-}` and process listings expose them. |
| Context/tool-output interceptors (`headroom`, `context-mode`) | Compressing tool output *before* it reaches the context window. Makes "return references, not payloads" infrastructural rather than a rule someone has to remember. Relevant to any agent that has been OOM-killed mid-run. |
| Mutation testing (`mutmut`, `cargo-mutants`, Stryker) | The single best answer to "is this test suite real?" — it is the automated form of *break it and watch it fail by name*. |

## Curated lists

- [awesome-harness-engineering](https://github.com/ai-boost/awesome-harness-engineering)
  — CC0. Sections on evals/verification and on security/sandbox/permissions are the
  relevant ones for this repo's concerns.

  **Read the repository, not articles about it.** One widely-shared summary of this
  list named five tools as its headline entries; four of them appear nowhere in the
  README, and the categories it described were not the list's categories. An
  AI-written aggregation of a curated list is two removes from the source and can be
  wrong about the *kind* of thing it describes, not merely the details. If a summary
  is why you are evaluating something, open the primary source first.

## Things that sound useful and were not

Recorded so the evaluation is not repeated:

- **`sqlfluff` as a correctness gate.** Run against a real migration it produced
  only `LT02`/`LT05` layout complaints — line length and indentation — and **zero**
  correctness findings. It would need substantial rule configuration to be worth
  wiring into a gate. Do not add it on the assumption that it catches SQL errors.

## Notes on tool identity

Small, costly ambiguities:

- **`yq` has two incompatible implementations.** The Python one (kislyuk, a `jq`
  wrapper) takes `yq '<jq filter>' file.yaml` and needs `-y` for YAML output;
  mikefarah's Go implementation uses different syntax entirely. Check which is
  installed before scripting against it.
- **`verapdf` is a Homebrew *formula*, not a cask.** `brew install --cask verapdf`
  fails. Verified against `formulae.brew.sh/api/formula/verapdf.json` (200) and
  `.../api/cask/verapdf.json` (404) — an example of checking a claim against the
  index rather than recalling it.

## Contributing tools

Useful entries include: the specific failure the tool addresses, its licence, and
whether *you* have run it. Entries marked "not evaluated" are welcome and honest;
entries implying evaluation that did not happen are the problem.
