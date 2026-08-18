# Verification: the specific ways a green result lies

Every item below is a defect that shipped or nearly did. They are grouped by the
shape of the lie, because the shape is what transfers between languages and
projects.

---

## 1. A check that cannot fail is decoration

**Prove every guard by negative control: break the thing, watch the check fail
BY NAME, restore, watch it pass.**

> **First, though: a check verifying a control nobody requires is ALSO
> decoration, however well it fails.** Applied without a stopping rule, this
> section manufactures guards for guards — we shipped a gate, then a check on the
> gate, then a check on *that*. Before proving a guard, cite the requirement that
> mandates the thing it guards. No citation means the candidate action is
> deletion. See [00 — the gate, and why we deleted it](./00-review-pipeline.md#the-gate-and-why-we-deleted-it).

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

### The negative control needs its own negative control

**A sabotage that does not change behaviour proves nothing about the guard.**

Breaking a rule is only evidence if the break would actually have mattered. A
denylist read `content/|legal/|disclosur|polic|terms|privacy|agreement|complian`.
To prove the test could catch a weakened rule, `content/|legal/` was deleted —
and every case still came out the same, because `terms`, `privacy` and `polic`
caught the same paths anyway. The suite reported "all passing" and that was
*correct*; the sabotage had changed nothing.

Replacing the whole pattern with one that matched nothing failed nine cases
immediately. That was the real control.

Before trusting a RED-verify, ask the same question you ask of a scan: **could
this break have produced a different answer?** A mutation inside a redundant
clause is a no-op, and a no-op mutation that "passes" is the same false assurance
as a scan that searched nothing — arrived at from the opposite direction.

Two cheap habits:

- **Delete the whole rule, not one clause of it.** If the guard still passes, the
  rule was never load-bearing and that is itself a finding.
- **Check the exit status, not the printed output.** `harness | tail` reports the
  status of `tail`, so a failing suite reads as exit 0. This was hit *in the same
  session that documented it*, while verifying a guard against exactly this class.
- **A flag computed and never read.** An invariant block set `ok = False` on each
  failure, printed `FAIL`, never referenced the variable again, and had no
  `sys.exit`. It exited 0 with any number of failures, so "require invariants
  green" was a human reading stdout. Same class as the pipe above, reached from
  the opposite direction: the status was computed correctly and then discarded.

## 1b. A rate computed over a mixed population measures the population, not the control

**This section exists because its first draft was wrong, was caught by review,
and was retracted.** The retraction is the finding.

### The claim that was nearly published here

A reusable adversarial-review workflow is called by **14 repositories**:

```bash
gh search code --owner <org1> --owner <org2> \
  "ci-workflows/.github/workflows/ai-review.yml" --limit 30 \
  --json repository,path --jq '.[]|"\(.repository.nameWithOwner)\t\(.path)"' | sort -u
# 16 hits, 2 of which are inside the workflow repo itself -> 14 callers
```

Counting **run outcomes** across six of those callers:

```bash
for r in <the six callers>; do
  gh run list -R "$r" --workflow=ai-review.yml --limit 100 \
    --json conclusion --jq '[.[].conclusion]|group_by(.)|map("\(.[0])=\(length)")|join(" ")'
done   # summed across the six by hand
```

```
cancelled 52 (54.7%)   skipped 35 (36.8%)   success 7 (7.4%)   failure 1
```

The draft concluded **"the control produces a review on one trigger in
fourteen"** and blamed `cancel-in-progress` plus a draft-pull-request gate. The
number was real; every inference drawn from it was wrong.

### Two measurements of the same thing, and they disagree

**Measurement 1 — count run outcomes.**

```bash
for r in <the six callers>; do
  gh run list -R "$r" --workflow=ai-review.yml --limit 100 \
    --json conclusion --jq '[.[].conclusion]|group_by(.)|map("\(.[0])=\(length)")|join(" ")'
done   # summed across the six by hand
```

```
cancelled 52 (54.7%)   skipped 35 (36.8%)   success 7 (7.4%)   failure 1
```

Conclusion drawn: the control is broken.

**Measurement 2 — split the population, because the denominator holds two
different kinds of thing:**

```bash
gh run list -R "$r" --workflow=ai-review.yml --limit 100 --json headBranch \
  --jq 'length as $t | ([.[].headBranch|select(startswith("dependabot/"))]|length) as $b
        | "\($b) of \($t)"'
```

```
33 of 34    32 of 33    16 of 17    5 of 6
```

Bot pull requests, which the workflow declines on purpose, are **86 of those 90
runs — about 95.6%** — in the four repositories checked.

Note the two ratios are different things, and conflating them is its own error:
`7.4%` is the **success rate** (7 of 95 runs), while `95.6%` is the **bot share**.
The success rate is low *because* the population is overwhelmingly work the
control is designed to decline. **`7.4%` was a fact about the traffic mix, not
about the control.**

That is the whole finding. What the *right* number is would need a different
measurement again — the unit this question is about is a pull request reaching a
verdict, and runs are not pull requests. **This section deliberately stops here
rather than asserting a coverage rate**, because an earlier draft asserted one
from run counts and was wrong.

**Not checked, and it matters:** the eight callers not inspected; whether the
non-bot runs correspond one-to-one with human pull requests; and the review
*quality*, which none of this measures at all.

And one more, found afterwards, which is the same lesson a third time: a later
pass looked for pull requests and found almost none — then a commit-level query
showed **22 agent-authored commits pushed straight to `main`** in two of those
repositories:

```bash
gh api --paginate "repos/$r/commits?since=<install-date>&per_page=100" \
  --jq '[.[]
        | select(.commit.author.email=="<the agent identity>")   # author, not just shape
        | select(.commit.message|test("\\(#[0-9]+\\)")|not)      # not a squashed PR
        | select(.parents|length==1)]                          # not a merge commit
        | length'
# 15 and 7 in the two repositories; 0 in the others checked
```
 **Absence of pull requests is not absence of work.** Wherever a
control is attached to one flow, measure whether the work is using that flow at
all before concluding the control is idle.

### A second instrument failure, in the attempt to fix the first

Scoping pull requests to a post-install window returned **0** for every
repository. That was not the answer — `gh --jq` does not accept `--arg`, so the
filter errored and a shell default rendered the error as `0`. A positive control
caught it: the same query with a date of `2020-01-01` also returned "0", which is
impossible.

**Run any filter with a value that MUST match before believing a zero.** And note
what nearly happened: a broken query agreed with the previous conclusion.
**Agreement with your last result is not confirmation when the new instrument is
silently failing.**

### The three errors, which are the transferable part

1. **A rate over a population you have not decomposed is not a measurement.** The
   denominator silently held two different kinds of thing. Before believing any
   rate, ask what is *in* the denominator and split it.
2. **The mechanism was invented, not measured.** The skips were blamed on a
   draft-PR gate. There were **zero** draft pull requests across 80 —
   `gh pr list -R "$r" --state all --limit 40 --json isDraft --jq '[.[]|select(.isDraft)]|length'`
   returns 0 on each. The condition had several branches and the one fitting the story was chosen
   without checking which fired. *Reading a condition is not measuring which
   branch of it fires.*
3. **Cancellation was counted as lost coverage.** A superseded run being
   cancelled does not stop the final commit being reviewed. Cheap cancellation
   and poor coverage are independent claims needing separate evidence — and
   neither query above measures what a cancelled run cost, so the draft's
   assertion that cancellation was the expensive part had nothing behind it
   either way.

### The check that would have caught it immediately

**Measure the outcome on the unit you actually care about.** The unit is a pull
request reaching a verdict, not a workflow run. Runs are what the API returns
easily; branches are what the question is about. **Wherever the convenient
denominator is not the unit of interest, this error lives.**

**How it was caught:** three independent CLI reviewers were given the draft plus
the source documents. Two returned REJECT and both said, independently, that
conclusion counts could not establish the causal story; one named the internal
contradiction — the same rate cannot be "the design working correctly" and
"evidence the control is absent". A fourth reviewer, given the raw API access
rather than the write-up, produced the branch-level decomposition that killed the
claim outright. **In this instance no single lane found everything**: one named
the contradiction, another the unsupported mechanism, and the decomposition came
from the reviewer given raw query access rather than the write-up.

**Not checked:** whether coverage holds outside the 100-run window; whether the
`skipped` conclusions are all bot traffic in the eight callers not individually
inspected; and the review *quality* — this measures only that a verdict was
produced, never whether it was any good.

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

### A pattern that cannot match returns zero and reads exactly like clean

The most common form of this is not an unset variable. It is a **search term that
was never in the text.**

A document said *"so does the base **branch** advancing"*. The sweep looked for
`base advancing`. Zero hits, reported as "the class is closed" — and the sentence
shipped unchanged. Review found it again the next round, in the same file, at the
line that had just been declared clean.

The failure is invisible because the output is identical to success. Three
defences, in order of cost:

1. **Sweep on the shortest distinctive stem**, not the phrase you remember. `advanc`
   near `base` finds every inflection; `base advancing` finds one.
2. **Positive-control the pattern**: confirm it matches at least one known
   occurrence before trusting a zero. If it cannot find the instance you are
   holding, it will not find the ones you are not.
3. **Grep for the concept, then read**, rather than grepping for the exact wording
   and trusting the count. Wording varies; the concept does not.

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

The same subsection covers a variant worth naming, because the control looks even
more convincing: **right question, right plumbing, wrong subject.** An exclusion
filter kept sensitive files out of a large upload and was verified properly —
unfiltered they appeared, filtered they were gone. But the filter matched
*source* paths, while the upload runs against a **staged** tree whose paths an
earlier step rewrites. The rule could never match anything it would be shown; the
control had been run against the source tree, verifying a deployment that would
never execute. The exclusion held anyway, through an unrelated upstream step — so
the outcome was safe and the control was theatre, which is the combination that
teaches misplaced confidence. **Run the control against the artifact the check
will actually be given**; if a pipeline rewrites paths, a path-based control is
only meaningful downstream of that rewrite. *Not checked in that incident: how
many other controls in the same pipeline were verified against the pre-transform
tree — the audit was stopped after the first one was found.*

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

## 4b. Proportion the evidence to the surface, and report only what you ran

Prior art: `AGENTS.md` in `deepseek-ai/deepseek-harness` (read 2026-08-17). Three
of its rules are sharper than anything we had written, and none of them is about
that project specifically.

- **"Report only commands run."** Not "tests pass" — *these* commands, this
  output. The gap between the two is where a plausible summary of a run that
  never happened lives.
- **Match evidence to the surface.** Focused tests for behaviour, snapshots for
  user-visible output, doc checks for docs, real end-to-end only for the
  integration boundary. **"Never default to the full suite or repeat a passing
  check."** A full-suite reflex reads as rigour and is mostly cost; it also
  trains everyone to skip verification when it is expensive, which is exactly
  when it matters.
- **Distinguish a blocked check from a failed one.** When a command fails because
  the environment denied credentials, network or IPC, that is **not** evidence
  about the code. Their rule: *require sandbox evidence, and never bypass a
  genuine failure or the sandbox under test.* A blocked check reported as a
  failure sends you debugging the wrong thing; reported as a pass, it is a check
  that cannot fail.

## 4c. Give time-bound guidance an expiry condition in its own text

Also from that file, and the most transferable idea in it. Its opening section is
titled **"Pre-release stance: foundation over blast radius"** and its first
sentence is **"Remove this section at the first tagged release."**

The section states the condition under which it stops being true, inside itself.
Nobody has to remember; a reader at the first release knows the paragraph is
spent. Compare a rule that was written for a temporary situation and simply
stays — which is how a document becomes confidently wrong about current state,
the failure this whole repo exists to catch.

**If guidance is contingent, write the contingency into it.** "Until X ships",
"while Y is unavailable", "remove at the first Z". A rule with no expiry
condition is claiming to be permanent, and most are not.

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

## 7b. Fix the class, not the line the review cited

**A review names one occurrence. The defect usually lives in several.** Fixing
only the cited line is how a single finding becomes three rounds.

Measured on one pull request: a documentation claim was wrong in four files.
Round one cited file A, which was fixed and the class declared closed. Round two
cited file B. Round three cited a *second sentence in file B* that the sweep for
round two had missed. Each round cost a full adversarial review, and every fix
was individually correct.

The reviewer is not being pedantic by re-raising it. It is reporting what it can
see, one instance at a time, because that is what a diff shows it.

**The habit:** when a finding is about a claim, a name, a constant or a pattern —
anything that can appear more than once — the fix is a *sweep*, not an edit. State
the sweep and its result in the adjudication, so the next round can check the
sweep rather than re-find the instance.

Three traps inside the sweep itself, each of which has produced a false "clean":

- **The pattern must be able to match** (see §2 above — this is the same defect
  wearing a different hat, and the two compound: an incomplete fix verified by an
  unmatchable sweep reports success twice).
- **Hidden files are excluded by default** in some tools. One sweep returned a
  confident 11 hits and missed the file holding 7 more — the dotfile whose entire
  purpose was the thing being swept for.
- **Case matters.** A case-sensitive sweep missed a variant in a change whose
  subject *was* case variants.

**Not everything that matches should change.** A dated record — a changelog entry,
a decision log, a release note — describes what was true when written. Rewriting
it to match current behaviour makes the history wrong in order to fix a document
that was not making a present-tense claim. Sweep the whole repository, then decide
per hit; say which ones you deliberately left.

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

Four more, each paid for with a real incident:

- **`rm -rf` on a name that matches nothing exits 0**, and then verifying the
  deletion *with the same name* confirms itself. **Verify a deletion by size,
  inode or a directory listing — never by the name you deleted with.**
- **Editing a script while it is running.** bash reads the file by byte offset,
  so truncating and rewriting in place (`>`, `write_text`) makes it execute the
  wrong bytes from the edit point on — usually a partial line, occasionally a
  valid but different command, and it **exits 0**. `sed -i` replaces the inode
  and is safe; the convenient habits are the unsafe ones.
- **`pkill -f <pattern>` matches the shell running `pkill`**, because the pattern
  is in that process's own command line. A cleanup that kills its own session
  looks exactly like the cleanup working. Use `pgrep` first, kill by PID, and
  never pattern-kill from inside the process you are matching.
- **A `while read` loop feeding a subprocess that also reads stdin.** `ffmpeg`
  consumes the remainder of the loop's input; the loop processed 4 of 56 files
  and exited 0. Pass `-nostdin` (or `< /dev/null`), and **reconcile the output
  count against the input count** — the exit status will not tell you.
