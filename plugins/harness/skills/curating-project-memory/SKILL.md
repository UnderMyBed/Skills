---
name: curating-project-memory
description: Use when the same correction has come up more than once, when a session ends and something was learned that should outlive it, when asked to "remember this" or "add that to CLAUDE.md", when a memory file has grown long enough that rules are being ignored, or when a rule that was written down is being violated anyway.
---

# Curating Project Memory

A rule that gets written down and then violated anyway is the normal case, not the
exception. Across 20,574 real sessions, agents violated an explicitly stated
instruction in **38% of episodes overall and 49% in CLI harnesses**, and only
**3% of those episodes self-corrected** (arXiv:2605.29442). Writing the rule is
the cheap half. Making it hold is this skill.

## The rule that decides everything else

**Never write a rule without first deleting the instruction that made the old
behavior correct.**

Most recurring corrections are not drift. They are the agent faithfully following
an instruction that is still sitting in the repo. A real example, from a project
where a "no narrative in docs" rule had to be taught twice three days apart:

> "this wasn't drift. Two files actively instruct the milestone record to exist —
> `docs/README.md:22` and `CLAUDE.md` points at it. **Every closeout that added
> narrative was correctly following instructions.** So the plan's last task
> replaces both instructions with a gate; without that, the restructure lasts
> until the next milestone."

If you add the rule and leave the contradiction, you have not fixed anything. You
have created a conflict, and conflict is the thing that actually degrades
compliance: follow rate falls from **96% to 20%** across twenty stacked
instructions, driven by reproducible pairwise conflicts (arXiv:2608.02639).

## Do not gate on file length

The common limits are folklore and following them costs you real rules.

- "Frontier models follow ~150–200 instructions" misreads IFScale §4.6, which says
  *primacy bias peaks* around 150–200 — where failure is most selective, not where
  capacity ends. The paper recommends no budget (arXiv:2507.11538).
- The only controlled test of memory-file structure — 1,650 agent sessions, 16,050
  AST-verified observations — found **no effect of file length**. At 25 / 100 /
  250 / 500 lines compliance ran 60.0 / 65.2 / 67.7 / 64.0%; p=0.16, and
  **BF₁₀ = 0.096**, which is affirmative evidence *for* the null. Instruction
  position within the file: p=0.83 (arXiv:2605.10039).

What actually moves compliance is contradiction, and attenuation within a session
— the median position of the first ignored instruction is the **fourth**
generated unit of work. Neither is fixed by a shorter file.

One real hard limit does exist and is often confused with the folklore: an
auto-memory index is read only to its **first 200 lines or 25 KB**, and everything
past that is silently dropped.

## Capture cheaply, promote expensively

Capture must be near-free or it stops happening. Promotion must be deliberate,
because unbounded self-authored memory is *worse than none at all*: measured
across 87 tasks and 18 configurations, curated skills scored **+16.6 points** over
a no-skills baseline while **self-generated ones scored 8–11 points below it**,
producing "confidently incorrect procedural advice" (arXiv:2602.12670).

So: let observations accumulate unreviewed in whatever staging your harness
provides. Promote only in a deliberate pass, and only what clears the bar.

### The promotion bar

Promote an observation only if **all four** hold:

1. **It recurred**, or violating it once already cost real work.
2. **It is not derivable** from the code, the config, or the tool's own defaults.
   If reading the repo answers it, delete it — that content measurably does not
   help, and repository overviews specifically were found unhelpful while adding
   over 20% to inference cost (arXiv:2602.11988).
3. **It does not contradict** anything already in memory.
4. **Removing it would cause a mistake.** If the agent already does the right
   thing without it, it is noise.

### Route it to the layer that can enforce it

A learning is not automatically a memory-file line. Pick the weakest layer that
actually guarantees the outcome:

| The learning is… | It belongs in | Not |
|---|---|---|
| A judgment to apply case by case | a memory-file rule | a hook |
| A command that should never prompt | the permission allowlist | a sentence asking nicely |
| Something that must happen **every time** | **a hook** | a rule you hope fires |
| A reusable technique with a recognizable trigger | a skill | more memory-file text |
| A condition that must break the build | a test or gate | a doc |
| Context only this subtree needs | a path-scoped rule file | the root file |

The distinction that matters most: **memory is advisory, hooks are enforcement.**
"Never edit `.env`" in a memory file is a request. A hook that blocks the edit is
a guarantee. Anything you have written down twice and seen violated twice is
telling you it was never a memory-file item.

## Procedure

**1. Gather.** Read the staging area and the session's own history. Look for
correction turns, the same instruction given twice, a gate that got bypassed, and
anything that cost a retry loop.

**2. Cluster.** Group observations that are the same underlying rule wearing
different words. Three phrasings of one preference is one rule, not three.

**3. Test each cluster against the four-point bar.** Most candidates die here.
Killing them is the point — this is where bloat pressure is absorbed.

**4. For each survivor, run the full promotion:**

```
a. SEARCH for the instruction that made the old behavior correct.
   Grep the memory files, the README, the docs index, the templates,
   the contributing guide. It is usually there and usually load-bearing.
b. DELETE it.
c. WRITE the rule, positively. State what the output IS, not a list of
   what to avoid — in head-to-head wording tests, prohibition phrasing
   produced MORE of the unwanted behavior than a positive recipe, and
   trended worse than no guidance at all.
d. GATE it, if it belongs in a gate. A rule with a mechanical check
   survives; a rule without one lasts until the next person reads the
   thing you forgot to delete.
e. COMMIT all of it together, so the rule and the deletion cannot
   drift apart.
```

**5. Reconcile.** Read the whole memory file top to bottom once. You are looking
for pairs that cannot both be satisfied. Fix them now — a conflict left in place
degrades every other rule in the file, not just its partner.

**6. Reload.** A memory file is read once at session start and held. Editing it
mid-session does not apply — the edit lands, and nothing changes. Finish by
clearing context or restarting, or the work you just did is invisible until the
next session.

## Worked example

A session where the agent twice ran `git commit --no-verify` to get past a
pre-commit hook that shelled out to a missing binary, disclosed it both times, and
was asked "don't we have a pinned toolchain?"

**Naive fix** — add to CLAUDE.md: *"Don't use `--no-verify`."*

That fails. It is a prohibition, it is advisory, and it leaves the actual cause —
a hook depending on a binary the environment does not guarantee — in place. The
next session hits the same wall and either bypasses again or stalls.

**Full promotion:**

```
a. SEARCH  → .husky/pre-commit calls `pnpm lint`, but nothing installs pnpm.
             CONTRIBUTING.md says "run npm install to get started" — the
             instruction that made an unpinned toolchain correct.
b. DELETE  → that line goes.
c. WRITE   → CLAUDE.md: "The toolchain is pinned in mise.toml. Run
             `mise install` before anything else; every gate assumes it."
d. GATE    → two of them, because the rule has two failure modes:
             - permissions.deny: "Bash(git commit --no-verify*)"
               so bypassing is not available rather than discouraged
             - pre-commit checks the pin is present and fails with the
               fix command, instead of dying on a missing binary
e. COMMIT  → all five changes in one commit, message explaining that the
             bypass was a symptom of the unpinned toolchain, not laziness.
```

One observation, one deleted instruction, one rule, two gates. The rule now
survives someone reading CONTRIBUTING.md, and the bypass is structurally
unavailable rather than merely discouraged.

## Red flags

| Thought | Reality |
|---|---|
| "I'll add the rule now and find the contradiction later" | Later never comes. The contradiction is why it recurred. |
| "The file is getting long, I should trim first" | Length is not the problem. Contradiction is. Trim what is derivable, keep what is earned. |
| "This is obviously right, it doesn't need a gate" | Every rule that had to be taught twice was obviously right the first time. |
| "I'll write it as 'never do X'" | Prohibitions measurably underperform positive recipes. Say what the output is. |
| "I'll capture this one properly instead of staging it" | Deliberating at capture time is how capture stops happening. Stage first, judge in the pass. |
| "I'll let the agent decide what's worth keeping" | Self-generated memory scored *below* having none. The gate is the whole value. |
| "Edited the file, we're good" | It does not apply until the session restarts. |

## Related

- `harness:uplifting-agentic-setup` — when the whole setup has gone stale, not
  just memory.
- `harness:bootstrapping-a-project` — establishing memory in a repo that has none.
