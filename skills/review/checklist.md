# Review checklist

- **Correctness.** Off-by-one, inverted condition, wrong variable, missing `await`/`?`, a branch
  that cannot be reached, a loop that never ends.
- **Errors.** An error swallowed, logged and ignored, or turned into a default that hides it.
  `unwrap`/`!` on a value that can be absent.
- **Edges.** Empty input, one element, very large input, Unicode, a path with spaces, a missing
  file, a concurrent caller.
- **Callers.** A changed signature or behaviour — is every caller still right? A removed check
  leaves its siblings in the other call sites unprotected.
- **Concurrency.** Two callers on one piece of state, a lock taken in the wrong order, a read
  outside the lock that should be inside, an `await` between a check and the act it guards.
- **Resources.** A process, file, thread, connection, subscription or temporary directory that
  is opened and not closed on every path — including the one that throws.
- **Security.** Input from outside reaching a shell, a query, a path or HTML unchecked. A path
  from the client joined onto a root without canonicalising it. A secret in a log, an error
  message or a commit.
- **Data.** A write that is not atomic where a crash could leave it half-done, a schema or
  format change with no way back, a migration that runs twice.
- **Tests.** Does a test fail if this change is reverted? A test that passes either way checks
  nothing.
- **Leftovers.** Debug output, commented-out code, a feature flag or a TODO that should have
  been decided now.
