---
description: Add a CHANGELOG.md line for the change in the working tree
argument-hint: "[what the user can now do]"
---
Add one line to `CHANGELOG.md` at the repository root for the change in the working tree.

- Look at `git diff` and `git diff --staged` to see what changed.
- Put the line under `## Unreleased`, newest on top. Create the file with a `# Changelog` title and
  an empty `## Unreleased` if it is missing.
- Format: `- <today's date, YYYY-MM-DD> <what the user can now do>`, one sentence, from the user's
  side — not which files changed.
- A refactor, a test or an internal fix the user never sees gets no line: say so and stop.

$ARGUMENTS
