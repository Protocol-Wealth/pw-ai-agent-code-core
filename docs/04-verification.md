# Verification: the specific ways a green result lies

Every item below is a defect that shipped or nearly did. They are grouped by the
shape of the lie, because the shape is what transfers between languages and
projects.

---

## 1. A check that cannot fail is decoration

**Prove every guard by negative control: break the thing, watch the check fail
BY NAME, restore, watch it pass.**

Four guards written in one night could not have reported the condition they
existed for:

- a `404` treated as degradable, which CI converts to a pass — and setting the
  "strict" flag did not fix it, because **both catch branches still exited 2**.
  *Check the exit path, not the flag.*
- a test that skipped on **any** error (`catch { return; }`), so a broken template
  passed against the exact defect it was written for. Narrow skips to the specific
  environment fact.
- a generator that swallowed a failure, exited 0, and left a **stale artifact** for
  the next check to compare against.
- a query naming two columns that did not exist, where **the mock shared the
  query's wrong assumption** and agreed with the bug.

Two more of the same family, worth naming because they look nothing alike:

- A gate testing that a value was **non-empty**, where the caller always supplied a
  locally-computed fallback for it. The gate's own error text said "fail closed";
  it could not fire, because the thing it tested was unconditionally populated.
  **Generalise this one:** any guard of the form `if (!x) block()` is only as
  strong as the guarantee that `x` can actually be absent. Check the callers, not
  the guard.
- A drift check that treated *"the canonical path is a directory"* as a transport
  failure and exited with a degradable status. A directory listing is a
  **successful, well-formed answer** meaning the file is gone. Structural answers
  must fail loudly; only genuine ambiguity may degrade.

**The general rule:** if a guard has a degraded/warning path, ask what fraction of
real failures land in it. If the answer is "most of them", it is not a guard.

## 2. A scan that searched nothing looks like a scan that found nothing

**Every negative result needs a positive control.** Before trusting "0 findings",
prove the scan *could* have returned one.

- **Shell state does not persist between tool calls.** An unset path variable makes
  `grep` search nothing and report `0` — indistinguishable from a clean scan. Set
  and use a variable in the same call.
- `cd` *does* persist as a tool's working directory in some harnesses, so a later
  command silently runs in the wrong repository. A failed `cd` in an `&&` chain
  short-circuits the rest, so a following heredoc never writes.
- **Reconcile counts on every batch job**: compare output count to input count.

### The subtler version: a positive control that proves the wrong thing

This is the one that gets past careful people.

Searching a codebase for background work, the query was
`setInterval|node-cron|scheduleJob` — zero matches across 542 files. A positive
control was run (`setTimeout` matched three files) and passed. The conclusion —
"no background work" — was wrong: the actual pattern was fire-and-forget work
started *during* a request and continuing after the response
(`void doWork(...).catch(...)` before `return response`).

**The control proved the scan was reading the tree. It could not prove the scan
was asking the right question.** When a negative result matters, have someone
adversarially attack the *query*, not just the plumbing.

### The same lesson from the other direction: check precision, not just non-emptiness

Contributed independently, and it sharpens the rule. A keyword scan over a
2,729-row corpus, looking for mis-titled items, returned **5 rows**. Four were
false positives — a legitimate article *about* bot-detection frameworks, and three
astronomy pieces matching on the word "Forbidden". **Precision: 20%.**

A positive control would have passed cleanly. The instrument was live and returning
rows; the question was wrong.

> **A positive control proves the instrument is live. It never proves the question
> is right.**

**A well-named API can be the narrowing.** A census over 29,442 photos used
Pillow's `Image.getexif()` and reported the `DateTimeOriginal` tag on 3 files.
That call returns **IFD0 only**; the tag lives in the Exif sub-IFD behind pointer
`0x8769`. Re-read with a library that walks the full structure, 183 of 190
hydrated JPEGs carried it — 96%, `piexif.load(p)["Exif"][36867]` over the sample.
The instrument was live and returned real EXIF for every file, from the wrong half
of the structure, and **"absent" and "not traversed" share one return value.**
Unlike a `--include` flag, the narrowing is not visible at the call site to be
questioned. Choosing an API is choosing a query: §4 of this document applies —
the governing source is the file's APP1 segment, and nobody opened one file.

So a positive control needs a complement: **adjudicate a sample of what the scan
returned.** Non-emptiness is not evidence of correctness in either direction — an
empty result can mean the query is wrong, and a non-empty result can be 80% noise.

## 3. Mocks agree with the bug

A unit test whose fixture was written from the same assumption as the code proves
nothing about the world. Where correctness depends on an external contract — a
schema, a view, a wire format, a validator's behaviour — **assert against the
artifact that defines it**, not against a hand-written double.

Corollary: **test the failing path, not only the missing path.** A control matrix
covering "tool absent" and "tool succeeds" is not a control for "tool present and
reports failure" — which is exactly where the cleanup code lives.

## 3b. Mutation-test the TESTS, not only the code

If the organising claim is "a check that cannot fail, reporting success", then the
checks most worth attacking are the tests themselves.

Three assertions written in one small file, by an author *specifically trying to
prevent this*, could not fail:

- A test asserting a rate was correct **passed with the pre-fix code fully
  restored** — it happened to run inside the date window the bug was scheduled to
  escape.
- Its purity check, `fn.length === 1`, was defeated by giving the parameter a
  **default value** — defaults do not count toward `Function.length`.
- A test named *"takes the maxima INDEPENDENTLY"* **never called the function under
  test**; it asserted `Math.max` over a local literal.
- A "no date-dependence" test that only ever ran at the current date.

Two were caught by review, one only by mutation.

**The rule: before trusting a new test, break the thing it covers and confirm it
goes red.** If the covered behaviour is time-dependent, *move the clock* rather
than trusting today's reading. In one session ~12 such mutations were run, and
every one that failed to go red was a real gap.

Tools that automate this — `mutmut`, `cargo-mutants`, Stryker — are the mechanised
form of the same discipline.

## 4. Verify a claim against the source that governs it

Not a summary, not recall, not another agent's report.

- a regulatory citation → against the rule text
- a schema claim → against the migration
- a hash → against the shipped bytes
- a package's install method → against the package index API
- "this validator requires X" → by running the validator

Two corrections in one night came from skipping this: a rule paragraph that was the
right letter and the **wrong entity**, and a diagnosis of a broken tool that turned
out to be an unset variable.

**A current value is not a historical one.** Date a configuration by its file
mtime plus the command log, never by reading today's value and attributing it to a
past event.

**Window selection is part of the claim.** One measurement peaked at 89.5% over 30
days and 66% over 7. A one-week read would have rejected a change that was
justified. State the window with the number.

## 5. Counts in prose are claims with an expiry

Write the command, not the number. ">100 clauses" was written twice from memory
when the measured values were 95 and 68. If a count must appear in prose, put the
command that produces it next to the number — or make a test assert it.

## 6. Prior notes are dated claims, not facts

Anything a previous session wrote — a handoff, a decision record, a status document
— asserts what was true when written. One state document described a resolved
outage as live for ten days.

If a record names a file, a flag, a count or a version, **verify it still holds
before acting on it.** This includes notes you wrote yourself an hour ago.

## 7. Classify findings before fixing

**Blocking / inaccuracy / cosmetic.**

The cosmetic fixes are where regressions come from. Across ~30 review findings,
~4 were cosmetic and those passes introduced new defects.

And a claim in a commit message is a claim like any other. Twice, a message
asserted a fix was applied everywhere when it had been applied in one of three
places — caught only because a reviewer checked the claim against the tree rather
than believing it.

## 8. Shell-specific traps that produce silent success

Collected because they recur and each one silently converts failure into success:

- `set -euo pipefail` + `grep` that matches nothing → **exit 1 kills the script**
  mid-way. A completed run reports itself as failed, or dies before its final step.
- `producer | head -c N` under `pipefail` → producer gets SIGPIPE, the pipeline
  fails, and the script dies **after** writing a partial file that looks complete.
- `cmd | tail` in a wrapper → the pipeline's exit status is `tail`'s. **A crashed
  job reports success.** Check `PIPESTATUS`.
- `trap "rm -f '${var}'" EXIT` → double quotes interpolate at definition time and
  the body is re-parsed as shell. A filename containing a quote **executes**.
  Use a function: `cleanup() { rm -f "${var:-}"; }; trap cleanup EXIT`.
- A second `trap ... EXIT` **replaces** the first. Two owners of one global
  resource; the first one's cleanup silently stops running.
- `${var:-}` on a variable you never declared reads the **environment**. An
  inherited value can make a cleanup routine delete an arbitrary path.
- `date -d` is GNU-only; `mktemp --suffix` is GNU-only. On BSD/macOS these do not
  misbehave, they fail — so a cross-platform claim in a runbook is a claim to test.
- Command substitution assignment (`x="$(cmd)"`) propagates the failure under
  `set -e`. Fine when intended; fatal when the command legitimately returns
  non-zero (a validator reporting non-conformance).
