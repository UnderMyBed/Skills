---
name: uplifting-agentic-setup
description: Use when returning to a repo that has been idle for weeks or months, when a repo's agent setup feels behind the ones you work in daily, when the same friction keeps recurring in one project but not others, when asked whether a project is "still current" or wants a "tooling uplift", or after a long gap in which the agent tooling itself has moved on.
---

# Uplifting an Agentic Setup

A repo you have not touched in months is running the harness you built for the
tooling that existed then. The gap is rarely in the code — it is in the setup
around it: permissions that were never scoped, gates that no longer run, memory
that describes a structure the repo has since outgrown, and mechanisms that did
not exist when you left.

This skill is a diff and a repair, not an audit report. It holds no stored
baseline and no version stamps. Current practice is re-derived every run, from
what is actually installed and shipping now — which is what makes it still correct
after a gap of any length.

## Do not hand-roll the parts that ship

Before assessing anything, run the tooling that already exists. Reinventing these
is the most common way this skill wastes an hour:

| Already shipped | Covers | Do not rebuild |
|---|---|---|
| `/doctor` | trims memory files by cutting what is derivable — directory layouts, dependency lists, architecture overviews — while keeping pitfalls, rationale, and non-default conventions; dedupes local against checked-in; migrates always-loaded guidance into skills | the memory-file trim |
| the permission-prompt analyzer | mines this project's transcripts for read-only calls that keep prompting, and proposes a scoped allowlist | hand-writing allowlists from memory |
| `plugin validate` / `plugin tag` | manifest correctness; and that a plugin manifest agrees with its marketplace entry | custom manifest linters |
| the plugin details view | per-component always-on vs on-invoke token cost | guessing what a plugin costs |

Run them first. Assess what they do not cover.

## Procedure

### 1. Detect

Read the repo's current state before forming any opinion. You are gathering, not
judging:

- memory files, and any path-scoped rule files
- the settings file, the permission allowlist, and any hooks
- the gate command, and whether it runs at all
- toolchain pins, and whether a fresh clone could satisfy them
- CI workflows, and when they last passed
- git state: branch, unpushed work, open PRs, open issues
- the agent tooling version, and what the repo's config predates

### 2. Assess against the ten dimensions

Diff what you found against `references/well-formed-repo.md`. Score each dimension
present / partial / absent. Do not repair yet — a dimension that looks absent is
sometimes handled somewhere you have not read.

### 3. Check the version delta

This is the step that makes "uplift to current" mean something without stored
state. Establish roughly when the repo's setup was last touched, then ask what has
shipped since that the repo should now be using.

Do not answer this from memory — your training data is precisely the stale source
here. Read the changelog and the current documentation. Mechanisms appear, change
their defaults, and get superseded on a timescale of weeks; a repo idle for four
months can easily be behind on isolation, verification, review configuration, and
orchestration limits simultaneously.

For each candidate mechanism, the test is not "is this new" but **"does this repo
have a friction that this removes?"** New is not a reason.

### 4. Repair, most-costly-friction first

Order the work by what actually costs turns in *this* repo, not by dimension
number. The ordering that usually holds:

1. Anything that makes the agent ask permission for pre-authorized work.
2. Anything that makes a session start with a long rediscovery.
3. Anything that lets work end un-integrated.
4. Anything that makes a gate skippable.
5. Everything else.

Apply repairs as small, separately-reviewable changes. A setup uplift that arrives
as one large commit is unreviewable, and this is exactly the work where a wrong
change is invisible until it costs a session.

### 5. Verify

An unverified repair is not a repair. For each one, produce evidence:

- the gate command actually runs, and actually fails when it should
- the allowlist actually silences the prompt it was added for
- the hook actually fires — check it in a real session, not by reading it
- a fresh clone can reach a green gate using only what is written down

State plainly what you could not verify. "Installed but unexercised" is an honest
result; "done" is not.

## What staleness actually looks like

The signals are behavioral, not cosmetic. Ranked by what they cost:

| Signal | Reading |
|---|---|
| Permission prompts for commands this project runs constantly | allowlist never scoped to this repo — often it was scoped to a *different* repo and silently does nothing here |
| Session opens with a long rediscovery sweep before any work | orientation was never automated; you are paying it every single time |
| Unpushed branches, stacked work, nothing in review | no mechanism forces integration; the run ends where the plan ends, not where the work lands |
| A rule in memory that the repo visibly violates | something else in the repo still instructs the old behavior — see `harness:curating-project-memory` |
| Memory file describes a structure that no longer exists | derivable content that has rotted; `/doctor` cuts this class |
| A gate exists but is bypassed in the history | the gate depends on something the environment does not guarantee |
| Config references a mechanism that has since been superseded | version delta — step 3 |

## The trap

**Do not uplift a repo you are not about to work in.** This skill changes settings,
hooks, and gates. Applied speculatively across every idle project, it produces a
large surface of untested configuration whose failures surface months later, in a
session where you have no memory of having changed anything.

Uplift the repo you are opening, when you open it, and verify it in that session.

## Red flags

| Thought | Reality |
|---|---|
| "I'll write the audit report first" | The deliverable is a repaired repo. A report is the failure mode. |
| "This mechanism is new, let's adopt it" | New is not a reason. Match it to a friction this repo actually has. |
| "I know what's changed since then" | Your training data is the stale source. Read the changelog. |
| "I'll batch all the repairs into one commit" | Unreviewable, and wrong changes stay invisible until they cost a session. |
| "The hook is written, so it works" | Hooks fail silently. Fire it in a real session or it is unverified. |
| "Let me fix all the idle repos while I'm here" | Untested config, deployed broadly, failing later. Uplift what you are opening. |

## Related

- `harness:curating-project-memory` — when memory specifically is the problem.
- `harness:bootstrapping-a-project` — the same ten dimensions, installed from
  nothing rather than repaired.
