---
name: implement-fix
description: "Implement a piece of work based on a spec or set of tickets."
argument-hint: "Which issue/ticket to implement"
disable-model-invocation: true
---
 
Implement one piece of work, test-first, then review before committing.
 
1. **Get the target.** Read the issue/ticket the user names. If they didn't
   specify one, ask — do not assume which issue they mean.
2. **Agree the seams**, then invoke the `/tdd` skill to build. Run typechecking
   and single test files regularly as you go.
3. **Not done until reviewed.** Run the full test suite, then invoke the
   `/code-review` skill and address its findings. Do not commit before
   `/code-review` has run and major bugs have been fixed.
4. Commit your work to the current branch.
