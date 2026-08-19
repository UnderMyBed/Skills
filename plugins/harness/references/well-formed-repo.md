# The Well-Formed Agentic Repo

Ten dimensions. `bootstrapping-a-project` installs them, `uplifting-agentic-setup`
diffs and repairs them, `curating-project-memory` keeps 1, 2, and 10 true.

Each dimension gives a **detect** signal (what to look for), an **assess** bar
(what good looks like), and a **repair**. Nothing here is stack-specific — the
mechanisms are named by what they do, because the tools change faster than the
requirement does.

---

## 1. Memory

**Detect** — Is there a memory file? Read it top to bottom. Grep the repo for other
files that instruct behavior: contributing guides, doc indexes, templates,
per-directory rule files.

**Assess**
- No two instructions that cannot both be satisfied. Contradiction is the failure
  mode that matters — follow rate degrades from 96% to 20% across twenty stacked
  instructions, driven by reproducible pairwise conflicts.
- Nothing derivable from the code. Directory layouts, dependency lists, and
  architecture overviews measurably do not help and add over 20% to inference cost.
- Every line passes: *would removing this cause a mistake?*
- **Not** a line budget. The only controlled test of memory-file length found no
  effect between 25 and 500 lines (BF₁₀ = 0.096, affirmative evidence for the null).

**Repair** — Delete derivable content first; it is the largest and safest cut. Then
resolve contradictions, which means finding the *other* instruction, not just
rewording this one. See `harness:curating-project-memory` for the full promotion
procedure.

**Watch for** — Content that must survive context compaction belongs in the root
memory file. Path-scoped rule files and per-directory memory are dropped at
compaction and do not return until a matching file is read again.

---

## 2. Standing authorization

**Detect** — Read back through recent sessions. Count how many turns were the human
approving something they had already approved in principle. Look for repeated
one-word replies: variations of "yes", "go ahead", "merge it", "do it".

**Assess** — The agent proceeds unasked through the whole routine cycle for this
project. It pauses only for actions that are destructive, irreversible, cost money,
or reverse a decision the human made.

**Repair** — Write the authorization as a positive statement of what the agent may
do, not a list of what not to ask about. Name the cycle explicitly and end it at a
concrete state, so "done" is unambiguous.

**Watch for** — This is the single most expensive dimension when absent, and the
cheapest to fix. It is also the one most often written as a vague sentiment
("be autonomous") rather than an enumerated grant, which does not work.

---

## 3. Toolchain

**Detect** — Can a fresh clone reach a working state using only what is written
down? Try it. Check whether tool versions are pinned or inherited from whatever
happens to be installed.

**Assess** — Versions pinned in a file, one command that satisfies the pins, and
every gate assuming that command has run.

**Repair** — Pin, then make the pin the documented entry point, then make the gates
depend on it. A gate that depends on an unpinned binary fails in a way that looks
like the gate is broken rather than the environment.

**Watch for** — When an agent bypasses a gate, the cause is usually here. The gate
depended on something the environment did not guarantee, the agent hit the wall,
and routing around it was the only way forward. Fix the pin, not the agent.

---

## 4. Gates

**Detect** — Is there one command that runs everything? Run it. Does it pass? Does
it *fail* when it should — introduce a deliberate error and confirm it goes red.

**Assess** — Format, lint, typecheck, and tests behind a single command. It runs
fast enough to be run habitually. It cannot pass while the thing it checks is broken.

**Repair** — Consolidate to one entry point. A gate nobody can remember how to
invoke is not a gate.

**Watch for** — A gate that has never been proven to fail is not known to work. This
is the most common silent defect in the whole checklist: a gate that certifies
builds it never actually exercised.

---

## 5. CI

**Detect** — Do the same gates run automatically? When did they last pass? What does
a run cost in time and in billed minutes?

**Assess** — The local gate and the automated gate are the same gate. Cost is
bounded and known.

**Repair** — Make automation call the same entry point as dimension 4, so they cannot
diverge. Where cost matters, cache aggressively and scope triggers narrowly rather
than reducing coverage.

**Watch for** — Divergence between local and automated checks is a slow failure: work
passes locally, fails remotely, and trust in the gate erodes until people stop
reading it.

---

## 6. Permissions

**Detect** — Which commands prompt for approval in this repo, repeatedly? Check
whether the allowlist is scoped to *this* repo — an allowlist scoped to a different
path silently does nothing here, which looks identical to having none.

**Assess** — The commands this project runs constantly do not prompt. Everything
consequential still does.

**Repair** — Use the tooling that mines actual transcripts for repeat offenders
rather than writing entries from memory; you will both miss real ones and add ones
that never fire.

**Watch for** — Any environment description attached to permissions describes a
specific repo. Check it names *this* one. A stale description is worse than none,
because it asserts facts about the wrong project.

---

## 7. Hooks

**Detect** — Are there any? For each, when did it last fire, and how do you know?

**Assess** — Anything that must happen every time is a hook, not a written request.
Memory files are advisory; hooks are enforcement. The distinction is the difference
between *almost every time* and *every time without exception*.

**Repair** — Convert guarantees out of prose and into hooks. Anything written down
twice and violated twice was never a memory-file item.

**Watch for**
- Hooks fail silently. Reading one is not verifying it — fire it in a real session.
- A hook that reasons needs a lifecycle point that can actually block and that
  allows enough time. Points that fire during teardown typically cannot block and
  have their output discarded.
- A blocking hook needs a loop guard, and hosts generally cap consecutive blocks.

---

## 8. Session lifecycle

**Detect** — How many operations happen at session start before the human's second
turn? If a session routinely opens with a long rediscovery sweep, that cost is
being paid every single time.

**Assess** — Orientation is automatic: current branch, work in flight, open review,
open work items, uncommitted state. Handoff has a known, ignored location so nobody
improvises one and nobody accidentally commits it.

**Repair** — Automate orientation at session start. Give handoff a conventional
ignored path.

**Watch for** — Repeated failed attempts to commit scratch documents mean the
convention exists in someone's head but not in the ignore file.

---

## 9. Work tracking

**Detect** — Where does outstanding work live? Is there more than one place? Can the
agent read it directly?

**Assess** — One system, readable by the agent, where "what is next" is answerable
without asking a person.

**Repair** — Consolidate. Two trackers means neither is trusted.

**Watch for** — Work tracked inside prose documents cannot be queried, tends to
accumulate progress narrative, and is the usual reason a documentation set slowly
becomes a changelog.

---

## 10. Verification

**Detect** — Search recent sessions for completion claims. For each, was evidence
produced, or was it asserted?

**Assess**
- "Done" has a definition in this repo, and it is written down.
- Claims arrive with evidence: the command, its output, the artifact.
- Something other than the author checks the work. The writer grading itself is the
  weakest possible arrangement.
- For work built against a visible test suite, some criteria are held back. Agents
  saturate the suite they can see; the gap against held-out criteria grows sharply
  with project size.

**Repair** — Write the definition of done. Require evidence in the same message as
the claim. Add an independent check — a separate reviewer that starts from the diff
with no investment in the approach.

**Watch for** — Adversarial review over-reports by construction: a reviewer asked to
find gaps will find some even when the work is sound, and chasing every finding
produces over-engineering — extra abstraction, defensive code, tests for
impossible cases. Pair any review mechanism with a **disposition rule**: what
severity earns a change, how many minor findings are worth reporting, and that
re-reviews stop raising new trivia.

---

## Using this list

**Score present / partial / absent before repairing anything.** A dimension that
looks absent is sometimes handled somewhere you have not read yet.

**Repair by cost, not by number.** The ordering above is a checklist, not a
priority. Priority is whatever is costing this project turns right now — most
often 2, then 8, then 10.

**Verify each repair in the session that made it.** Configuration that was installed
but never exercised fails later, in a session with no memory of the change.
