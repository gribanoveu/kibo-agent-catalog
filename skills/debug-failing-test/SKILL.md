---
name: debug-failing-test
description: Find out why a test fails and fix the cause — reproduce it alone, read the whole failure, decide whether the code or the test is wrong, narrow it down, fix, and prove it. Use when a test, a test suite or CI is red, or a test is flaky.
---

# Debug a failing test

1. **Reproduce it alone.** Find the command that runs just this test (the project's test runner with a
   name filter) and run it. Note the exact failure. It passes alone but fails in the suite → it depends
   on order or shared state: run its neighbours with it.
2. **Read the whole failure** — the assertion, expected against actual, and the first frame of the
   stack trace that is in this project's code, not the framework's.
3. **Decide which side is wrong.** Read the test and the code under it. The test encodes the intended
   behaviour unless it is clearly out of date (the requirement changed, and the commit that changed it
   says so). If it is not clear which side is right, ask rather than pick.
4. **Narrow it down.** A small hypothesis at a time: a log line or a debugger at the point where the
   values first go wrong, not everywhere. It used to pass → `git log` on the files involved, and
   `git bisect run <test command>` when the range is long.
5. **Flaky** (passes and fails with no change) → run it many times in a loop and look for time, order,
   randomness, network, shared files or ports, unawaited async work. Fix the race; a retry or a sleep
   is not a fix.
6. **Fix the cause** with the smallest change that does it, then run the test, its file, and the
   suite of the module touched.
7. Report: the cause in a sentence, the fix, and the commands run with their result.

Never make a test pass by weakening its assertion, skipping it, or catching the error it reports —
unless the user agrees the test itself is wrong.
