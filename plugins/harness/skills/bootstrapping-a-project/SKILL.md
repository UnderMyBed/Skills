---
name: bootstrapping-a-project
description: Use at the start of a new project or a rebuild, before the first real code — when a repo is empty or nearly so, when someone says "let's build X" and there is nothing to read yet, when scaffolding a new service or subsystem, or when a greenfield session is about to fan work out across agents.
---

# Bootstrapping a Project

Project zero is the worst regime an agent works in, and the failure is measurable.
On thirty build-from-scratch tasks spanning 1,500 to 110,000 lines, **every
frontier agent saturated the visible test suite on every task** while diverging
sharply from a held-out suite — and the gap grows roughly **28 percentage points
per tenfold increase in code size**. More search made it worse, not better
(arXiv:2605.21384). A separate run found agents scoring near-perfect against a
hidden oracle while the delivered library was, on inspection, "dead or absent"
(arXiv:2606.28430).

Greenfield *is* the large-new-codebase regime. That is why the ground has to be
laid before the building starts, not after.

## Do not start with a codebase-analysis step

Tools that generate a memory file by analyzing the repo have no referent when
there is no repo. They variously hang on an empty directory, read files from
unrelated projects nearby, or produce an architecture overview — which is exactly
the content that memory-file trimming later removes, because it is derivable and
measurably does not help (arXiv:2602.11988).

Write the memory file from the interview in step 2, by hand, short. Every line
must pass: *would removing this cause a mistake?*

## The scale gate

Run this first, and let it set everything downstream. Full ceremony on a small
project is a measured loss — one comparison put a heavyweight spec-driven flow at
roughly **10× the wall-clock of ordinary iterative prompting, producing 2,577
lines of markdown for 689 lines of code**.

| Scale | Signals | Ceremony |
|---|---|---|
| **Throwaway** | answering a question, a spike, discarded within the day | none. Skip to writing code. Label it throwaway and mean it. |
| **Small** | one person, one surface, no consumers, reversible | steps 1, 3, 4. Skip the written spec; a paragraph of intent is enough. |
| **Real** | will be maintained, has users or consumers, someone else will run it | all seven steps. |

A new project is **architectural** regardless of how familiar the domain feels —
there is no existing flow to read, so nothing about it is bounded. Familiarity
with the *kind* of app is not familiarity with *this* app.

## Procedure

### 1. Prior art, before any stack decision

Survey what already exists — libraries, standards, and any predecessor or
comparable implementation — before choosing anything. Do this first, because it is
the decision everything else is downstream of, and it is the one most expensive to
revisit.

The failure this prevents is specific: building for a day or more, then
discovering the thing you were building already exists and works better, and
throwing the work away.

Where a predecessor codebase exists, mine it deliberately rather than copying or
avoiding it. Classify every inherited finding:

- **Platform facts** — how the underlying system actually behaves. Verify once,
  then trust.
- **Experience observations** — "this approach was slow." n=1. Cheap to re-test;
  do not inherit as law.
- **Patches on wrong approaches** — a fix for a problem the old architecture
  created. *If your architecture never produces the disease, do not import the
  cure.* This is the largest category and the most tempting to copy.

Evaluate ideas provenance-blind in both directions. Rejecting an idea *because*
the predecessor used it lets a mediocre predecessor steer you just as much as
copying it would.

### 2. Interview, then spec, then a fresh session

Interview before writing anything. Ask about the hard parts — the constraints,
the edge cases, the things that would make this project different from the
obvious version of itself. Do not ask what you can infer.

Then write the spec. A spec is self-contained when it names the files and
interfaces involved, states what is explicitly **out of scope**, and **ends with
an end-to-end verification step that proves the feature works**.

The failure this prevents: a spec that covers implementation but silently omits
whole classes — design, naming, hosting, domains, launch, cost — which then
surface one at a time, each forcing a rewrite. Before declaring the spec done,
walk the classes explicitly and record "not applicable" where it is.

Execute in a **fresh session**. Time spent making the spec precise pays back more
than time spent watching the implementation.

### 3. Install the harness before the first feature

This is the step that gets skipped, and skipping it is what turns design work into
a stalled project. See `references/well-formed-repo.md` for the full checklist;
the minimum before any feature code:

- **Toolchain pinned**, with one command that makes a fresh clone runnable.
- **One gate command** that runs format, lint, typecheck, and tests together.
- **The gate wired to run automatically** — not as a request in a memory file.
  The gap between "run the linter" written down and the linter running is the gap
  between *almost every time* and *every time without exception*.
- **Bypasses closed.** If the gate can be skipped with a flag, block the flag.
  Agents route around broken gates rather than reporting them.
- **Permissions scoped to this repo** for the commands it actually runs.
- **A memory file** written from step 2, short, with nothing derivable in it.

### 4. Walking skeleton before any parallelism

Build one end-to-end path a **human** can exercise. Not a module, not a layer —
a thin vertical slice that runs.

> "In the era of LLM coding, it has never been easier to get the entire system
> working. By using the system, it becomes obvious what the next steps should be.
> **The LLM can't dogfood the code it writes.**"

That is the whole argument. The skeleton exists so a person can use the thing and
form an opinion, because the agent structurally cannot.

It is also the gate on fanning out. Parallelizing before a spine exists
*manufactures* the disconnection that integration then has to hunt — components
that are each individually correct and do not share state. That is the dominant
greenfield failure mode, ahead of outright bugs.

Do not proceed until the skeleton runs green.

### 5. Hold back the oracle

Reserve a slice of the acceptance criteria the agent building the work cannot see.
Verify against the held-out slice before calling anything done.

Without this, you are measuring against the target the work was optimized toward,
and the measurement above says that number will look excellent and mean very
little. This is the single highest-value greenfield practice and the one most
often absent.

### 6. Verify dependencies exist

Check that every package you are about to install is real, at the version named,
before installing it. Hallucinated package names remain a live supply-chain
surface — one study found 127 package names invented *identically* by five
independent frontier models, of which 53 were still registrable
(arXiv:2605.17062); another measured 27.75% hallucinated version recommendations
across 36,870 real enterprise dependency upgrades.

Also pin versions and pull current documentation rather than relying on recall.
Models reproduce deprecated API patterns for 70–90% of outdated functions, and
retain them even when handed the current spec.

### 7. Then parallelize

With a green skeleton, a working gate, and a held-out check, fan out. Not before.

## Why the order is the order

Errors at project zero lock in early and stay. Across 1,794 agent trajectories and
over 63,000 execution steps, the decisive error landed at **median step 7**, the
recovery window averaged **one step**, failure signals surfaced roughly ten steps
later, and **82% of failed runs kept executing past the point of no return**
(arXiv:2607.09510).

Verification cannot be added at review time. It has to already be there when the
run starts.

## Red flags

| Thought | Reality |
|---|---|
| "I know this kind of app, this is straightforward" | Bounded measures the repo, not your familiarity. An empty repo has no flow to read. |
| "Let's get something working, we'll add tests after" | Held-out divergence grows with code size. Later is the most expensive possible time. |
| "The tests pass" | Against the suite the work was optimized toward. That is the number the research says is uninformative. |
| "I'll set up CI once there's something to run" | The gate is what makes the skeleton meaningful. It comes first. |
| "Let's fan out, there's lots to build" | Before a spine exists, parallelism manufactures the disconnection you will spend longer fixing. |
| "The scaffolding tool will write the memory file" | It writes an architecture overview — the content that measurably does not help. |
| "This library is probably called that" | 53 registrable names invented identically by five models. Check. |
| "Prior art can wait, let's start" | It is the decision everything is downstream of and the most expensive to revisit. |

## Related

- `harness:curating-project-memory` — keeping the memory file true as the project runs.
- `harness:uplifting-agentic-setup` — the same ten dimensions, repaired rather than installed.
- `misc:prior-art` — the full method for step 1.
- `conductor:coordinate-agents` — step 7, once the skeleton is green.
