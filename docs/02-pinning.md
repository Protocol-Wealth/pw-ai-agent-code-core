# Pinning: when a content hash belongs on a document, and when it is wrong

Getting this backwards is expensive and the cost is not obvious up front. Pinning
one document that should not have been pinned produced a version-repin treadmill
across three repositories, and each repin generated its own defects.

This applies well beyond regulated work: the same distinction governs an open-source
project's licence file, a freelancer's contract, and a company's privacy page.

---

## The distinction

Every published document is one of two things, and which one decides how it is
versioned.

| | **Contract** | **Disclosure / notice / informational** |
|---|---|---|
| Examples | Advisory agreement, terms and conditions, terms of service, an SOW | Privacy policy, security posture page, subprocessor list, risk disclosures, a status page |
| What it does | Binds two parties **on the terms in force when signed** | States **current practice, as of now** |
| Pinned? | **Yes** — name, effective date, content hash | **No. Never.** Referenced; the archive preserves what it said on any date |
| Why | Posting a newer file must not silently amend a signed agreement. Amendment needs consent from both parties | Pinning asserts a per-recipient frozen regime you do not operate. The archive already supplies the history |

**The test is what the document does, not what it is called.** A file named
"policy" may be a notice; one named "terms" may be neither. Check what the
governing rule or the relationship actually requires, and record the citation next
to the decision.

A document that binds nobody does not become a contract by being important.

## The corollary that catches people

**A contract may reference a notice. It must not pin one.**

If an agreement pins the privacy policy to a hash, then every routine privacy
update becomes a contract amendment requiring consent — and you have asserted that
each signer is governed by a frozen privacy regime, which is almost certainly not
how you actually operate.

Reference it by name and address. Let the archive hold what it said on any given
date.

### The same trap, one level down

Cross-references between two notices are also pins. If document A says
*"Companion to: Privacy Policy (effective August 13, 2026)"*, then republishing
the privacy policy on the 14th strands A's reference — and A cannot be corrected
in place, because published editions are immutable.

Reference companions **by name and URL**, never by effective date. Each document
keeps its own effective date as its own identity; what goes away is one document
asserting a specific *edition* of another.

## Integrity is not incorporation

These are different mechanisms and conflating them reinstates the mistake:

- **Archival integrity** — hashing every published edition so you can later prove
  a document was not altered after the fact. **Good practice for everything**,
  including notices.
- **Incorporation by pin** — a signed instrument naming a specific hash, so the
  signer is bound to that exact text. **Contracts only.**

"Maintain dated, archived, cryptographically verifiable versions" is a statement
about the first. It is not an instruction to pin notices into agreements.

## Pinning is the last act of publication, never a step during drafting

A hash computed over text that is still moving is wrong the moment anything else
changes — and a stale hash is **worse than no hash**, because it reads as verified.

Draft with the hash absent or explicitly marked as a placeholder. Compute it once,
over the final bytes, as the last step. Then verify that what the document *states*
is what the shipped file actually *hashes to*.

Four corollaries, each of which has cost real time:

1. **Hash the bytes the recipient receives.** Not a re-render, not a pre-metadata
   copy. Render once, hash that, ship that.
2. **A hashed render must be reproducible.** Pin any build clock to the document's
   own effective date. Otherwise the same edition hashes differently on every
   build and no two recipients hold the same identifier. *Check this directly:
   render twice, `sha256sum` both.* Most document toolchains embed a creation
   timestamp by default.
3. **Identity is (name, effective date), never a version number.** Intermediate
   version strings are usually not retained, so they identify nothing anyone can
   look up.
4. **One date, one text.** If the text changes materially after publication, the
   edition changes. Two texts under one date breaks the only guarantee the hash
   provides.

## Verifying a pin, in one command

The check is trivial and is skipped constantly. What the document *says* its hash
is, versus what the shipped file *is*:

```bash
sha256sum path/to/shipped.pdf
grep -o '[a-f0-9]\{64\}' path/to/manifest-or-agreement
```

If those disagree, the document identifies a file nobody has.

## A guard worth having

If a manifest pins both a hash and metadata (an effective date, a version label),
note that the hash **cannot** protect the metadata: swap the file, update the hash,
forget the date, and every check still passes because the hash is over the new
bytes and matches perfectly.

Add a check that the stated effective date appears **inside** the document — and
prove it by negative control: change one date by a single day and watch the check
fail by name.
