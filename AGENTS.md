# AGENTS.md — pw-ai-agent-code-core

This repository publishes evidence-backed practices for operating AI coding
agents. Read `README.md` and `.agent-lead.yml` before changing a document.

## Scope

- Keep shared practices in `docs/`. Put domain-specific additions in `Personal/`,
  `Business/`, or `RIA/` according to `.agent-lead.yml`.
- Treat prior notes as dated evidence. Check a claim against the governing source
  before repeating it, and give the date and verification command for counts.
- Label an external source that has not been read or verified. Distinguish a
  tested tool from one that was only reviewed or listed.
- Use synthetic examples. Do not add client information, personal data,
  credentials, internal hostnames, or private repository material.
- Preserve the evidence and failure that motivated a rule when refining it.
  A new practice needs a concrete failure mode and a check that could fail.

## Working in this repository

- Follow path ownership in `.agent-lead.yml`. Shared files are co-owned; propose
  changes for the relevant leads to review.
- Check every local Markdown link you add and keep the README's document map in
  sync with actual files.
- Use `AGENTS.md` as the agent guidance entry point. Do not add `CLAUDE.md` or
  another agent-specific copy of these instructions.
