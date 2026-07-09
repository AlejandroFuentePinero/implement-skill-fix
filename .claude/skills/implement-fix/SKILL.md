---
name: implement-fix
description: "Implement a piece of work from a spec or tickets, test-first, through the /tdd and /code-review gates."
argument-hint: "Which issue/ticket to implement"
disable-model-invocation: true
---

Implement one piece of work, test-first, and pass it through two **gates** before it lands: `/tdd` on the way in, `/code-review` on the way out. A gate is a checkpoint the work passes through, not advice you weigh. Neither gate is satisfied by writing test-shaped or review-shaped code yourself; each is satisfied by invoking the skill and carrying its output forward.

## 1. Get the target

Read the issue or ticket the user names. If they named none, ask which one before starting.

**Done when:** you can state in one line the behaviour this work delivers and where it lives.

## 2. Agree the seams

Name the seams you will test at, highest seam first, and confirm them with the user before writing code. These seams are the contract the next step builds against.

**Done when:** the user has agreed a written list of seams. If a spec already fixed them in its Testing Decisions, restate them and confirm rather than re-deriving.

## 3. Build through `/tdd`

Invoke the `/tdd` skill and build each agreed seam through its red → green loop. Run typechecking and the single test file for the seam you are on as you go.

**Done when:** every agreed seam has a test that went red then green under `/tdd`. A seam with code but no red-then-green test is unbuilt.

## 4. Pass the `/code-review` gate

Run the full test suite. Then invoke the `/code-review` skill and work its findings until every major one is addressed.

**Done when:** `/code-review` has run against the diff and its findings are triaged, every major one fixed. This is the gate: the diff reaches step 5 only once it has passed here.

## 5. Commit

Commit the reviewed diff to the current branch. What lands is exactly what passed the gate.
