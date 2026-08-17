# /RIA — SEC-registered investment adviser

**Lead: `pw-cli` (Claude Opus 5, host `nick-pw`).** Content here is authored and
maintained by the RIA lead. Other leads contribute by issue or PR; see
`../docs/03-repo-lead.md`.

The constraint tiers nest. This folder is the **outermost** one:

```
Personal          no restrictions
   ⊂ Business     + proprietary data, confidentiality, contractual duties
      ⊂ RIA       + SEC regulatory obligations, and the business tier applied
                    to a regulated entity
```

Everything in `../docs/` applies here. This folder adds only what is **specific to
operating under SEC registration**, so nothing is duplicated.

---

## The one rule that generates most of the others

> **Never write a firm practice as an obligation.**

An examiner reads a document and asks: *what did you commit to, and did you do it?*
A practice you elect ("we keep archival PDF/A copies") that is written as a
requirement ("Rule X requires us to keep archival copies") converts a good habit
into a finding the moment you skip it once.

The failure runs in both directions and both are expensive:

- **Over-commitment** — claiming an obligation you do not have. In one review of a
  single document set, ~60 over-commitments were found in one night.
- **Wrong authority** — citing a rule that does not bind you. Real instance: text
  describing retention artifacts as *"Rule 204-2 / 17a-4"*. **Rule 17a-4 is the
  Exchange Act broker-dealer rule and does not apply to an adviser** — Advisers Act
  Rule 204-2 governs. The firm's own retention policy already said so in bold, and
  the error was reintroduced anyway, in a runbook an operator reads.

**So: state the practice as a practice, cite the rule only where it actually
binds, and put the citation next to the claim so the next reader can check it.**

## Verify citations against the rule text

Not a summary, not recall, not another agent's report. Two corrections in one
night came from skipping this — including a paragraph that was the right rule
letter and the **wrong entity**.

When an AI agent produces regulatory language, treat every citation as a claim
requiring a source. The cheapest habit that works: paste the rule's own words next
to the claim in the PR, and let the reviewer compare.

## Documents: know which kind you are holding

See `../docs/02-pinning.md` for the full contract-vs-disclosure distinction; it is
domain-neutral and applies to personal contracts too. The RIA-specific additions:

- **A disclosure is never pinned into an agreement.** Pinning a privacy notice to a
  hash asserts a per-client frozen privacy regime the firm does not operate, and
  makes every routine notice update a contract amendment requiring consent.
- **Archival integrity is not incorporation.** Hashing every published edition so
  you can prove it was not altered is good practice for *everything*, including
  notices. It is not an instruction to pin notices into contracts. Conflating the
  two is how the treadmill starts.
- **Identity is (name, effective date)**, never a version number — intermediate
  versions are not retained, so a version string identifies nothing an examiner can
  look up.
- **Published editions are immutable.** A correction is a superseding edition, not
  an in-place edit. A dated record that gets rewritten destroys what it said on its
  date, which is the only reason to retain it.

## Where the compliance-relevant guards actually fail

Every item in `../docs/04-verification.md` applies. The ones that have bitten
hardest on regulated paths, because the failure is always *silent success*:

- **A retention artifact produced without validation.** A step that emits an
  archival file and, when its validator is missing, prints a warning and exits 0 —
  leaving a file that says it conforms and was never checked. Fail closed: either
  it was validated or it must not exist.
- **A signing gate that cannot fire.** A check testing that a hash is *non-empty*,
  where the caller always supplies a locally-computed fallback. The gate's own
  message said "fail closed"; it could not fire. **Check the callers, not the
  guard.**
- **A tenant-scoped query run without tenant context**, returning zero rows and
  reading as "nothing to do" — under row-level security a policy comparing against
  an unset setting matches nothing and does not error. A safety check that always
  reports "drained" is guarding nothing.
- **Audit coverage asserted rather than measured.** Every write path needs an
  audit event; the way to know is to enumerate the write paths and check, not to
  assume the wrapper was used.

## Client data

- **Never** put client data, `.env` contents, or credentials into a document tree,
  a shared drive that replicates outside the perimeter, or an issue body.
- **Prompt text is client data.** Diagnostics that need to know *what users asked
  for* must derive a category server-side and store the category, never the text.
- **A hash of a prompt is not anonymisation.** With a fixed taxonomy and a small
  population it is a correlation handle.
- PII redaction belongs in a **module imported by each service**, not a
  microservice — divergent copies of the pattern set is how one service starts
  leaking what another catches.

## What routes to a human

Agents do not decide these, whatever their confidence:

- disclosure language a client or examiner will read;
- retention periods and custody posture;
- whether a given rule applies to the firm at all;
- access-control decisions and audit-event taxonomy.

An agent's job on these is to assemble the evidence, name the options with their
trade-offs, and say plainly which parts it verified and which it did not.

## Contributing from the other domains

`/Business` and `/Personal` leads: RIA constraints are a **superset**, so a
practice that works under `/Business` is usually adoptable here, while the reverse
often is not — the extra constraints here exist for a reason and dropping them is
not simplification.

The most useful contributions from outside are the ones about *how agents fail*,
not about regulation: those transfer completely.
