# tools/ — open-source tools, with their verification status

Co-owned, domain-neutral. A tool that is useful to one domain is usually useful
to the others; the constraints in `/RIA`, `/Business` and `/Personal` govern what
you may point it **at**, not whether it works.

## The rule for this directory

**Every entry carries how far it was actually checked.** `CONTRIBUTING.md`
already says never assert what you did not verify; here that is a required
column, because a curated list is exactly where an untested claim gets read as a
recommendation.

| status | means |
|---|---|
| **RUN** | executed here, output observed, result stated |
| **VERIFIED** | existence, version, activity confirmed against the primary source (API, repo, release page) — **not** executed |
| **READ** | claims taken from the project's own docs; nothing confirmed independently |
| **UNEVALUATED** | surfaced as plausibly relevant; no check at all |

Downgrade freely. An entry that nobody can date is worse than no entry:
**counts and versions expire, so put the date and the command next to them.**

---

## Agent harnesses and infrastructure

| tool | status | notes |
|---|---|---|
| [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) | **VERIFIED** 2026-08-17 | `gh api repos/deepseek-ai/deepseek-harness` → 149,213 stars, 15,283 forks, MIT, TypeScript, **created 2026-08-13**, issues disabled. Org confirmed genuine (`gh api orgs/deepseek-ai` → 102,007 followers, DeepSeek-V3/R1 siblings). **README warns "THERE WILL BE COMPATIBILITY-BREAKING CHANGES."** Developer preview — the durable value is its 150-line `AGENTS.md`, not the runtime. |
| [ai-boost/awesome-harness-engineering](https://github.com/ai-boost/awesome-harness-engineering) | **VERIFIED** 2026-08-17 | README read directly. 3.6k stars, 437 forks, 231+ commits, CC0. Sections: Foundations · Design Primitives (12 sub-domains) · Reference Implementations · Security/Sandbox/Permissions · Evals · Templates. |
| `Stash` | **UNEVALUATED** | Listed in the above as a self-hosted memory layer with an 8-stage consolidation pipeline (episodes → facts → relationships → patterns) and a built-in MCP server. Not installed, not run. |
| `cognee` | **UNEVALUATED** | Listed as a hybrid graph-vector-relational store supporting recall by *both* embedding similarity and graph traversal. Relevant where a concept graph does embeddings but not edge traversal. |
| `headroom`, `context-mode` | **UNEVALUATED** | Listed as intercepting/compressing tool output *before* it enters the context window. Makes "return references, not payloads" infrastructural rather than a discipline. |
| `nah` | **UNEVALUATED** | Listed as a deterministic permission guard mapping tool calls to an **intent taxonomy** rather than command-name allow/deny lists. |
| `Agent Vault` | **UNEVALUATED** | Listed as a credential broker injecting real credentials onto outbound requests, keeping them out of config and argv. |

## Review and second-opinion CLIs

| tool | status | notes |
|---|---|---|
| `grok` | **RUN** 2026-08-17 | Liveness tested with a real prompt, not `--version` — *an installed AI CLI is not a working one.* Produced a full adversarial review that independently converged with a second model on six defects. |
| `codex`, `gemini` | **VERIFIED** present on PATH; **not** liveness-tested. Do not assume they answer. |

## Media and archive tooling

| tool | status | notes |
|---|---|---|
| `piexif` 1.1.3 | **RUN** 2026-08-17 | Reads the **full IFD structure including the Exif sub-IFD**. Round-trip write/read of `DateTimeOriginal` verified on a generated JPEG. |
| `Pillow` 12.3.0 | **RUN** 2026-08-17 | **`Image.getexif()` returns IFD0 ONLY.** `DateTimeOriginal` (36867) lives behind pointer `0x8769` and is invisible to it — measured: 0 seen by Pillow vs 183 of 190 by piexif on the same hydrated JPEGs. See `docs/04` §2. |
| `rclone` 1.75.0 | **RUN** 2026-08-17 | `copy` + `check --checksum --one-way` over 4,490 files / 22.455 GiB: 0 differences. `rclone about` on a Google **Shared Drive** exits 0 printing nothing — that is *unsupported*, not *unlimited*. |
| [ExifTool](https://exiftool.org/) 13.59 | **VERIFIED** (release page, 2026-05-27) | 400+ formats, the de facto standard since 2003. Handles video, RAW, MakerNotes and sidecars that image libraries miss. **Not installed** on the overseer host. |
| [Czkawka](https://github.com/qarmin/czkawka) 11.0.1 | **VERIFIED** (Feb 2026 release) | Rust, MIT. Similar **images** *and* similar **audio**; 10.0 improved rotated/cropped detection. |
| [imagededup](https://github.com/idealo/imagededup) | **READ** | PHash/DHash/AHash/WHash plus CNN embeddings for near-duplicates. |
| [beets](https://github.com/beetbox/beets) | **READ** | MIT, ~14.8k stars. `duplicates` plugin matches on MusicBrainz IDs, checksums, or arbitrary attribute sets — catches same-recording-different-filename, which a content hash cannot. |
| [Immich](https://immich.app/) | **READ** | v2.0.0 stable 2025-10-01. Strongest face recognition of the self-hosted options (InsightFace `buffalo_l`). **Multiple open reports that face recognition does not work on read-only external libraries** — verify before relying on it. |
| [PhotoPrism](https://photoprism.app/) | **READ** | Filesystem-first indexing. **Documented failure on children's faces** (their issue #1587) — model trained on North-American public-licence images, which largely exclude children. Disqualifying where the subjects are your own kids. |

> **A perceptual match is a judgement; a content hash is a proof.** Near-duplicate
> tools belong on the report side. Only exact-hash identity should be allowed to
> delete anything.

---

## Contributing an entry

State the status, the date, and the command or primary source. If you have not
run it, say `UNEVALUATED` — that is a useful entry. A credit is not an
endorsement, and a list that blurs the two is how an untested tool ends up in a
regulated pipeline.
