---
description: Draft release notes for the changes since the last tag
argument-hint: "[version, or a range like v1.2.0..HEAD]"
---
Draft release notes for the next release.

- The range: the one named below, or from the latest tag (`git describe --tags --abbrev=0`) to `HEAD`.
  No tags yet → the whole history, and say so.
- Source, in order of trust: the `CHANGELOG.md` section for these changes (usually `## Unreleased`),
  then merged PRs in the range, then `git log --no-merges <range>`.
- Write for the people who use the software: group under **Added**, **Changed**, **Fixed** and
  **Removed**, one line each, what they can now do or what behaves differently. Leave out refactors,
  tests, CI and dependency bumps unless they change something a user sees. Call out breaking changes
  and anything that needs an action (a migration, a new setting) at the top.
- Show the notes; do not tag, publish or edit files unless asked.

$ARGUMENTS
