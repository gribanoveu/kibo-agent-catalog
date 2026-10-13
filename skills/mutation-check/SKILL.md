---
name: mutation-check
description: Check that the tests of new or changed code would catch a bug — break the logic one small edit at a time, run the tests, and treat every break that still passes as a missing test. Use after writing code with tests, or when asked whether the tests are good enough, for mutation testing, or to check test coverage of a change.
---

# Check the tests by breaking the code

Green tests only show the code passes them. This shows they would fail if the code were wrong.

1. **Start green.** Run the tests of the module and note the command and how long it takes.
   The working tree must be committed or stashed first: every mutant is undone from git, and nothing
   of the user's may be lost with it.
2. **Pick mutants** in the code that changed — the decisions, not the formatting. One edit each:
   - a condition negated, or replaced by `true` / `false`;
   - a boundary moved: `<` ↔ `<=`, `+ 1` dropped, a limit doubled;
   - a check, a validation, or an error path removed;
   - a call removed (a write, a cleanup, a rollback, a notification);
   - a value swapped for the default (`""`, `0`, empty list, `None`);
   - two branches or two arguments swapped.
   Ten to thirty is usual for a feature; more for code that guards security or data.
3. **For each mutant:** apply the one edit, run the tests, record caught (a test failed — name it) or
   survived, and restore the file from git (`git checkout -- <file>`) before the next one. A mutant
   that does not compile counts for nothing: make one that does.
4. **Every survivor is a missing test**, not a bad mutant — unless the change really cannot alter
   behaviour (then say why: it is equivalent). Write the test that kills it, then apply that mutant
   again and see it caught.
5. **Finish clean:** `git diff` shows only the new tests; no mutant is left in the tree.
6. Report a table — mutant, caught or survived, by which test — then the tests added for survivors.
