# Review checklist

- **Correctness.** Off-by-one, inverted condition, wrong variable, missing `await`/`?`, a branch
  that cannot be reached, a loop that never ends.
- **Errors.** An error swallowed, logged and ignored, or turned into a default that hides it.
  `unwrap`/`!` on a value that can be absent.
- **Edges.** Empty input, one element, very large input, Unicode, a path with spaces, a missing
  file, a concurrent caller.
- **Callers.** A changed signature or behaviour — every caller still right?
- **Security.** Input from outside reaching a shell, a query, a path or HTML unchecked. A secret
  in a log, an error message or a commit.
- **Tests.** Does a test fail if this change is reverted? A test that passes either way checks
  nothing.
- **Leftovers.** Debug output, commented-out code, a TODO that should have been done now.
