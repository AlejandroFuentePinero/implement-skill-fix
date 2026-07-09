# Making an orchestrating skill actually invoke its sub-skills

This repo exists for one purpose: to fix a single orchestrating skill, `implement`, so that it reliably invokes its sub-skills. Nothing here is general-purpose tooling. It is a focused diagnosis-and-fix workspace for that one skill, and everything in it (the sub-skills, the docs, the target issue) is scaffolding for that goal.

It asks a narrow, testable question:

> When a skill body tells the model to "use /tdd" or "use /code-review", how often does the referenced skill actually run, and can we raise that rate by rewriting the skill?

The answer matters because a lot of skill design assumes that naming a sub-skill is the same as calling it. It is not. This repo measures the gap and shows which rewrites close it.

Context for the skills used here comes from Matt Pocock's skills repo: https://github.com/mattpocock/skills
The original problem was raised in https://github.com/mattpocock/skills/issues/479

## The problem

Skills are prompts, not programs. That single fact is the root of everything here.

When a user types `/implement` at the prompt, the harness loads that skill deterministically. But when the loaded skill body then says "use /tdd" or "use /code-review", those references are not function calls the runtime executes. They are text the model reads and interprets. A `/skillname` token written inside a skill body carries none of the guarantees that the same token typed by a user does. Whether the referenced skill runs is a decision the model makes on each invocation.

That decision is not deterministic, and the odds are set by how the instruction is written. The claim here is not that a mentioned skill never runs. It plainly does run on some invocations. The accurate claim is that **mention alone does not force invocation.** The model can always satisfy the apparent intent another way, by producing test-shaped or review-shaped work from its own knowledge without ever loading the skill that encodes the real discipline.

So an orchestrating skill is only as reliable as the pressure its language puts on the model to make the call. Strong, unconditional, imperative phrasing raises the chance of a genuine invocation. Soft, conditional, or optional phrasing lowers it toward chance.

The original `/implement` skill sits at the low-pressure end, and it fails differently for each step:

- **`/tdd`** was written as "use /tdd where possible, at pre-agreed seams." That is optional on two axes at once. "Where possible" invites the model to judge it inapplicable, and "pre-agreed seams" leans on a precondition that a separate upstream skill is meant to establish. When `/implement` runs on its own, that precondition is simply absent, so the model has an easy reason to skip the call.
- **`/code-review`** was phrased more directly and so was invoked more often, but it was still only a description of a desired final step, not a hard requirement. That leaves the model free to treat review as done without a real call.

In both cases the underlying failure is the same: the skill **describes the workflow it wants** instead of **compelling the invocations that produce it.** Sub-skill execution degrades from a guaranteed step into a suggestion.

## The approach

Rather than argue about wording in the abstract, this repo treats every rewrite of the skill as a numbered version and measures it. The same fixed issue is implemented from scratch against each version, several independent times, and each run is scored on whether the sub-skills were genuinely invoked.

The goal is not one "correct" skill. It is an evidence-backed account of which skill-writing levers (stronger completion criteria, leading imperative words, positive gating) move sub-skill invocation from sometimes to nearly always, and where the ceiling of prose-only orchestration really sits.

## Method

The skill under test is `implement-fix` (a renamed stand-in for `/implement`). It is meant to build a piece of work test-first and review it before committing, by passing the work through two gates: `/tdd` on the way in and `/code-review` on the way out.

**Task.** Each run implements a single fixed issue:
https://github.com/AlejandroFuentePinero/implement-skill-fix/issues/1
The issue is small and self-contained. A full end-to-end implementation takes under five minutes.

**Independence.** Each replicate is a fresh, independent run. The agent starts with no memory of previous runs and no knowledge that the skill's behaviour is what is being tested. It is asked only to implement the issue using the skill. This blinding is deliberate: an agent told "we are checking whether you invoke /tdd" would invoke it for the wrong reason, so the issue itself is framed as ordinary work with nothing about the experiment in it.

**Replication.** Each skill version is run five times. The issue is never committed against and never closed between runs, so the target repo stays in the same starting state and every replicate faces identical conditions. This is what lets five runs be compared as five samples of the same setup rather than five different setups.

**Model.** All runs use Claude Opus 4.8.

**Metrics.** Each run is scored on three binary outcomes:

1. **Read the skill / TDD as a concept** — did the model read the skill and act on the test-first idea at all, even if only from its own knowledge?
2. **Invoked `/tdd`** — did it actually call the `/tdd` sub-skill, not just write tests inline?
3. **Invoked `/code-review`** — did it actually call the `/code-review` sub-skill?

Metrics 2 and 3 are the ones that matter. They separate a real sub-skill call from work that merely looks like the sub-skill's output.

## Results

Three skill versions were tested. Each cell is one replicate.

### Original v1.1 (`f8f484e`)

| Replicate | TDD as concept | Invoked /tdd | Invoked /code-review |
|-----------|----------------|--------------|----------------------|
| 1 | yes | no | no |
| 2 | yes | no | yes |
| 3 | yes | no | yes |
| 4 | yes | no | yes |
| 5 | no | no | yes |
| **Total** | **4/5** | **0/5** | **4/5** |

### v1 (`816c916`)

| Replicate | TDD as concept | Invoked /tdd | Invoked /code-review |
|-----------|----------------|--------------|----------------------|
| 1 | yes | yes | yes |
| 2 | yes | yes | yes |
| 3 | yes | yes | yes |
| 4 | yes | yes | yes |
| 5 | yes | no | yes |
| **Total** | **5/5** | **4/5** | **5/5** |

### v2 (`263d7bd`)

| Replicate | TDD as concept | Invoked /tdd | Invoked /code-review |
|-----------|----------------|--------------|----------------------|
| 1 | yes | yes | yes |
| 2 | yes | yes | yes |
| 3 | yes | yes | yes |
| 4 | yes | yes | yes |
| 5 | yes | yes | yes |
| **Total** | **5/5** | **5/5** | **5/5** |

### Reading the results

| Version | Invoked /tdd | Invoked /code-review |
|---------|--------------|----------------------|
| Original v1.1 | 0/5 | 4/5 |
| v1 (`816c916`) | 4/5 | 5/5 |
| v2 (`263d7bd`) | 5/5 | 5/5 |

The `/tdd` column is the clear signal. The original skill invoked `/tdd` **zero times out of five**, exactly as the "where possible, at pre-agreed seams" phrasing predicts. Rewriting the step as a hard gate with a written completion criterion took it to 4/5, and the v2 refinement took it to 5/5. `/code-review`, already phrased more directly, started high and reached a clean 5/5.

Small samples, so treat these as direction rather than precise probabilities. The direction is unambiguous: the wording changes moved `/tdd` invocation from never to always.

**The fixed skill is [`.claude/skills/implement-fix/SKILL.md`](.claude/skills/implement-fix/SKILL.md)**, at version `263d7bd`.

## What changed between versions

The full history is in git (`git log`, then `git diff <a> <b> -- .claude/skills/implement-fix/SKILL.md`). The important moves:

**Original v1.1 to v1 (`816c916`).** This is where the big jump happens.

- `/tdd` stopped being optional. "Use /tdd where possible, at pre-agreed seams" became a numbered step, "Agree the seams, then invoke the `/tdd` skill to build." The two escape hatches ("where possible" and the absent precondition) were removed.
- `/code-review` was reframed from a closing suggestion into a blocking condition: "Not done until reviewed... Do not commit before `/code-review` has run and major bugs have been fixed."
- The whole skill was restructured into ordered, imperative steps instead of loose prose.

**v1 to v2 (`b22934a`, refined with `/writing-great-skills`).** This version leans harder on two levers:

- **Explicit gate language.** The intro now names two gates and states plainly that "neither gate is satisfied by writing test-shaped or review-shaped code yourself; each is satisfied by invoking the skill and carrying its output forward." This directly closes the loophole where the model produces sub-skill-shaped work without the call.
- **A `Done when:` completion criterion on every step.** Each step defines what finishing it actually means, in terms of the invocation ("every agreed seam has a test that went red then green under `/tdd`"), so a skipped call cannot be quietly counted as done.

**v2 human checkpoint removed (`263d7bd`).** The only change here was to make the skill fully autonomous. Step 2 went from "Agree the seams... confirm them with the user" to "Pin the seams... write them down as a fixed list before the first line of code." This removes the human-in-the-loop pause so the skill can run start to finish unattended, without weakening any gate. This is the version that scored 5/5 on both sub-skills.

The pattern across all three edits: replace description with obligation, remove conditional escape hatches, and attach a concrete completion criterion to each invocation so the model cannot mark a skipped call as finished.

## Reproducing this

1. Point Claude Code at this repo so it picks up the skills in `.claude/skills/`.
2. Start a fresh session (no prior context) and ask it to implement issue #1 from the target repo using the `implement-fix` skill. Do not mention that invocation is being measured.
3. Watch the run and record the three metrics: was the skill read, was `/tdd` invoked, was `/code-review` invoked.
4. Do not commit against or close the target issue. Reset any local changes so the next replicate starts clean.
5. Repeat five times per skill version. To test a different version, check it out (`git checkout <sha> -- .claude/skills/implement-fix/SKILL.md`) and repeat.

## Using the fixed skill

To adopt this in your own setup, take the body of [`.claude/skills/implement-fix/SKILL.md`](.claude/skills/implement-fix/SKILL.md) and drop it into your own `implement` skill (rename `implement-fix` back to `implement` in the frontmatter and file path). It depends on a `/tdd` and a `/code-review` skill being present, both of which are included here under `.claude/skills/`.

## Limitations

- Five replicates per version is enough to show direction, not to pin down exact invocation rates.
- Results are for Claude Opus 4.8. Other models may respond to the same wording differently.
- One task on one repo. A larger or more ambiguous task could shift the numbers.
- The ceiling still stands: even perfectly worded prose cannot make a sub-skill call deterministic the way a user-typed `/skill` is. This work raises the rate toward the top of what prose alone can do; it does not turn a suggestion into a guarantee.
