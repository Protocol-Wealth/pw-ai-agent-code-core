# /Personal — individual work, no external restrictions

**Lead: `personal-lead` (Claude Opus 5).** This file is a scope statement written by the
RIA lead so the boundary is clear from day one. **The content is yours to write** —
treat everything below as a starting frame to accept, reject or replace.

The constraint tiers nest, and this is the innermost:

```
Personal          no restrictions                                    ← here
   ⊂ Business     + proprietary data, confidentiality, contracts
      ⊂ RIA       + SEC regulatory obligations
```

Everything in `../docs/` applies. What this folder does *not* carry is any external
compliance obligation — no regulator, no counterparty, no contractual duty of
confidence.

---

## Why "no restrictions" is not "no practices"

The interesting thing about this tier is that it isolates the practices that are
worth doing **for their own sake**, with no regulator to blame. Everything left
standing here is load-bearing on the merits.

Most of it is already at root, because the failure modes do not care whose data it
is: a check that cannot fail reports success in a hobby project exactly as it does
in a regulated one — it just costs less when it does.

## What plausibly belongs here

Prompts, not an outline you owe anyone:

- **Personal contracts and documents.** `../docs/02-pinning.md` is fully
  applicable — a freelance agreement is a contract, a personal site's privacy page
  is a notice, and pinning the second into the first is the same mistake at smaller
  scale.
- **Irreplaceable data.** The sharpest constraint at this tier is not disclosure,
  it is **loss**. Photos, correspondence, records with no vendor to re-issue them.
  A commit that exists on exactly one disk is a real risk here in a way it is not
  where everything replicates.
- **Provenance of derived claims.** When an agent dates, categorises or dedupes a
  personal archive, the derivation *is* the record. A statistic measured on the
  subset that had corroboration, then applied to the subset that did not, is a
  scope error that survives because nobody re-derives it later.
- **Agent autonomy.** This is the tier where an agent can be given the most latitude
  — so it is the right place to work out what latitude actually needs, and where a
  guard earns its cost rather than being imposed.

## What the RIA lead can offer

Little that is domain-specific — which is the point. The transferable material is
already at root. Two things worth naming:

- **Evidence discipline transfers completely.** "Verify a claim against the source
  that governs it" costs the same here and is just as often skipped.
- **Dated records are superseded, not edited** — as true for a personal archive as
  for a compliance record, and easier to get wrong here because nobody is watching.

## What flows the other way

This tier is where the coordination and harness work lives in practice — multi-host
topology, memory, dispatch, context management. Those are **not** personal-specific
and the estate's hardest agent problems surface here first. When they do, they
belong at root, not in this folder.

## Cross-domain

`/Business` and `/RIA` add constraints; they do not remove practices. A practice
that is worth doing here is usually worth doing there, while the reverse is often
over-commitment — adopting a regulated tier's machinery without the obligation that
justifies it.
