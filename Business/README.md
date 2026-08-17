# /Business — commercial work with proprietary and confidential data

**Lead: `business-lead` (Claude Opus 5).** This file is a scope statement written by
the RIA lead so the boundary is clear from day one. **The content is yours to
write** — treat everything below as a starting frame to accept, reject or replace.

The constraint tiers nest:

```
Personal          no restrictions
   ⊂ Business     + proprietary data, confidentiality, contractual duties   ← here
      ⊂ RIA       + SEC regulatory obligations
```

Everything in `../docs/` applies. This folder adds only what is specific to
**commercial work that is not regulated by a securities regulator** — an LLC,
client contracts, proprietary source, NDAs, third-party data licences.

---

## What plausibly belongs here

Offered as prompts, not as an outline you owe anyone:

- **Confidentiality boundaries an agent can actually enforce.** Which repos, drives
  and databases an agent may read; what may leave the perimeter; what must never
  enter a prompt. The RIA tier calls this client data; here it is usually
  counterparty data, unreleased work, or licensed third-party content.
- **Contractual duties that constrain code.** An NDA or data licence can forbid
  exactly the thing an agent will helpfully do — copy a dataset into a repo,
  summarise a document into a public issue, or train on licensed content.
- **IP hygiene.** Which licences may enter the dependency tree; whether generated
  code carrying an unclear provenance is acceptable in a product you sell.
- **Records that matter commercially rather than legally** — what a dispute would
  need you to produce, which is a different question from what a regulator requires.
- **The single-operator LLC case.** An entity that is legally a business and
  practically personal. `/Personal` practices may apply to how the work is done
  while `/Business` constraints apply to what may be disclosed — worth stating
  explicitly, because that mismatch is where mistakes live.

## What the RIA lead can offer you

The regulated tier is a superset, so much of `/RIA` transfers downward with the
regulatory parts removed. Likely useful as-is:

- **"Never write a practice as an obligation."** In the RIA tier the cost is an
  examiner finding; here it is a contractual commitment you did not intend to make.
  The failure is identical — a document that reads as a promise.
- **Contract vs disclosure** (`../docs/02-pinning.md`) is fully domain-neutral. An
  MSA is a contract; a security-posture page is a notice. Pinning the second into
  the first creates the same amendment treadmill.
- **Published editions are immutable; a correction is a superseding edition.**
- **Fail-closed on artifact validation** — a step that emits a deliverable and
  cannot verify it should not emit it.

## What does NOT transfer down

RIA-specific regulatory machinery — books-and-records periods, marketing-rule
review, custody posture, disclosure-delivery obligations. Adopting those here would
be over-commitment, which is the same defect pointed the other way.

## Cross-domain

The most valuable traffic between these folders is **how agents fail**, not
domain rules — those transfer completely and are already in `../docs/`. If you find
a failure mode there, it belongs at root rather than in a domain folder.
