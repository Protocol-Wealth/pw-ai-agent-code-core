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

### The right API, the wrong surface

A verification can query a real, authoritative, correctly-authenticated endpoint and
still not observe the rule it claims to have checked.

An estate doc recorded that seven repositories were protected, "verified against the
API" on a stated date. The verification used classic branch protection. The rule that
actually gates merges lives in a **ruleset**, which is a different endpoint:

```
$ gh api repos/OWNER/REPO/branches/main/protection -q '.required_status_checks'
                                    # empty

$ gh api repos/OWNER/REPO/rulesets -q '.[] | "\(.id)\t\(.name)\t\(.enforcement)"'
20606587        main    active

$ gh api repos/OWNER/REPO/rulesets/20606587 \
    -q '.rules[] | select(.type=="required_status_checks") | .parameters
        | "strict=\(.strict_required_status_checks_policy)"'
strict=true
```

**That is still not enough, and the gap is the same mistake one layer in.** A ruleset
named `main` is not thereby a ruleset that governs `main`: the name is a label, and the
scope lives in `target` and `conditions.ref_name`. An active ruleset can target tags, or
`refs/heads/release/*`, and report `strict=true` while the branch under discussion is
governed by nothing. Ask for the scope in the same breath as the rule:

```
$ gh api repos/OWNER/REPO/rulesets/20606587 \
    -q '"target=\(.target)  include=\(.conditions.ref_name.include)  exclude=\(.conditions.ref_name.exclude)"'
target=branch  include=["~DEFAULT_BRANCH"]  exclude=[]
```

Only now does `strict=true` say anything about `main`. A verification that reads a rule
without reading its scope has confirmed that a rule exists somewhere, which is a different
claim from the one it is being used to support.

One finding on this section is worth recording because it was **wrong**, and refuting it
took less time than applying it would have. A reviewer called the listing incomplete
without `includes_parents=true`, on the grounds that an inherited organisation or
enterprise ruleset would be omitted. The parameter exists; its default is `true`
("Include rulesets configured at higher levels that apply to this repository. Default:
`true`"), so the listing already includes inherited rules and adding the flag changes
nothing. Checked against the API reference before editing, which is the whole discipline:
a fix applied to a false premise injects a defect and manufactures another round.

What the finding does point at is real, though it is a different thing: because the listing
mixes repository rules with inherited ones, the response is the only place that says WHERE
a rule lives. Read `source_type` (and `source`) per ruleset, or you will look for a rule in
the repository settings that is defined an organisation above it.

Two consequences, and the second is the one that stings:

1. A change was justified on the grounds that a required strict check made a second CI
   run redundant. That justification was **correct**, and the evidence for it was
   invisible to the documented verification procedure. Right conclusion, unexamined
   mechanism.
2. Bypass actors live on the ruleset DETAIL endpoint — a third surface again, and one
   the classic endpoint knows nothing about. They are perfectly observable once you ask
   the right endpoint, which is the point: this is a *scope* failure, not an
   unknowability. The ruleset above carries three at `bypass_mode: always`, one being a
   `DeployKey` entry with `actor_id: null`, meaning *any* deploy key with write access,
   present or future:

```
$ gh api repos/OWNER/REPO/rulesets/20606587 \
    -q '.bypass_actors[]? | "\(.actor_type)\tid=\(.actor_id)\t\(.bypass_mode)"'
DeployKey       id=null always
Integration     id=1144995      always
Integration     id=1236702      always
```

**Not checked:** whether any deploy key currently holds write on that repository — on
the one inspected, `gh api repos/OWNER/REPO/keys` returned empty, so the exposure is
latent rather than live. The two integrations were not identified; resolving an app id
to a name needs `admin:org`, which the operator's token lacked.

The generalisation: when a protection or policy claim matters, enumerate **every**
surface that can express it, and re-verify after any platform migrates a feature from
one to another. "We checked the API" names a method, not a scope.

### A control that passes identically before and after is broken

The negative-control rule above says a control must be shown to fail. The sharper
version: **run it against the unfixed code too, and require the two runs to differ.**

Three controls in one session passed on both versions, and each would have certified a
fix that did nothing:

| what was being tested | why the control could not fail |
|---|---|
| a reaper that kills orphaned child processes on interrupt | the script re-execs itself from a temp snapshot, so the pid matched by name was the pre-exec wrapper; the real process was never signalled |
| a guard that flags a run which verified nothing | the chosen input already incremented that counter by another path, so both versions reported the same |
| a turn-budget truncation guard | the input made the process exit non-zero, so an **older** guard caught it and the new one never ran |

In all three the control was rewritten, not the fix. The tell is cheap and mechanical:
**if the before and after runs produce the same output, the control is wrong until
proven otherwise.** Only the third was noticed from the output alone; the other two
were found by explicitly running the old version.

### A marker the reviewed content can contain

A harness that decides an outcome by searching output for a sentinel breaks in **both**
directions once the reviewed material can contain that sentinel:

- A fixed refusal token (`DIFF_UNREADABLE`) failed a *good* review, on the very commit
  that introduced the literal — the diff contained it. Fixed with a per-run nonce
  generated after the diff is written, so reviewed content cannot contain it.
- The mirror image, in the same file months later: a completion guard required the
  string `VERDICT:` to appear. That string occurs in the harness's own prompt and
  comments, so a truncated review that quoted the file it was reviewing **passed**.

Three successive narrowings were each defeated by a case a stub reproduced in one run:
substring anywhere → the file contains it; anchored to line start → narration quoting
the required format sits at column 0; within the last N lines → a three-line truncation
puts the quote inside any small window. What held was **the last non-empty line**,
which narration cannot satisfy while still having something after it to say.

Rule: a sentinel must be unforgeable by the material under test — generate it per run —
and a *required* marker must be positionally constrained, not merely present.

**Positional constraint raises the cost; it does not make the marker unforgeable, and
saying otherwise is the same overclaim this section is about.** If reviewed content
quotes the marker at column 0 and the output truncates immediately after that quoted
line, the quote *is* the last line and a truncated review passes. The last-line rule
defeats every shape we actually observed, which is worth having, but the property it
buys is "harder to satisfy by accident", not "impossible to satisfy by accident."

The unforgeable version is the one already used for the refusal token, applied to the
completion marker too: a **per-run nonce**, generated after the reviewed material is
fixed, so no content under review can contain it. Ask for `VERDICT-<nonce>:` and the
question stops being positional. That the same file already contained the technique and
did not apply it to the second marker is itself the recurring shape — a lesson learned
in one place and not carried across the room.

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
