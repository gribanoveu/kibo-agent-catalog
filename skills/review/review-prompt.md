# Reviewer prompt

Hand this to a fresh agent that saw nothing of the session that wrote the change — that is the
point: it gets the code and the intent, not the author's reasoning. Fill in every `<...>` and
pass it as one message, in the foreground — the next step waits for the answer.

```text
You are reviewing a change in the repository at <absolute path>, branch <branch>. Change
nothing and commit nothing: read, run what proves a point, report.

The change: <`git diff <base>..HEAD`, or the uncommitted `git diff` plus `git status` for new
files>.

Read first:
1. The project's rules — AGENTS.md, CLAUDE.md, CONTRIBUTING, or whatever says how this code is
   written and layered.
2. <The plan, issue or ticket the change comes from, and the decisions it cites.>
3. The whole diff, and for each changed place the code around it, not only the diff lines.

Look for, in this order:
- Bugs: a wrong result, a race on shared state, a crash, an error swallowed or turned into a
  default, an unchecked value where absence is possible, a leaked resource (process, file,
  subscription, thread, cursor), a secret reaching a log, an error message or the disk.
- Rule violations: the layering and dependency direction the project states, a type from the
  framework in a layer that must not know it, a literal where the project has a token, a
  bypass of the project's own wrapper or helper, a synchronous call where the project requires
  the asynchronous one.
- Contract drift: what the change promises in its plan and does not do, or does differently
  without saying so.
- Tests: a rule of the new code that no test guards; a test that passes either way; a test
  double that does not behave like the real implementation.
- Extra: an abstraction, flag or parameter kept for a later that may never come; a
  re-implementation of a helper the project already has (name it); dead code left behind.

Run the project's checks — type-check, build, tests — and say what failed. If only one layer
is touched, that layer's checks are enough.

Verify every finding before you report it: trace the path from a real call or input to the
failure. If you cannot, say so and mark it unverified rather than dropping or inflating it.

Answer as a list, most severe first:
- `path:line` — blocker / bug / rule / test-gap / nit — what is wrong — the scenario it shows
  up in — what you suggest.

End with one line: what you checked and looks sound. If there is nothing to report, say so; do
not invent findings.
```
