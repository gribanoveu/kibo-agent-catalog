---
description: Commit the change in the working tree with a message in the repository's style
argument-hint: "[files, or what the commit is about]"
---
Commit the change in the working tree.

- Look at `git status`, `git diff` and `git diff --staged`. Commit what is staged; if nothing is,
  stage only the files the request below names, or the ones that belong to one change — never
  `git add -A` with unrelated files in the tree: list them and ask.
- Match the message to `git log --oneline -15` and to any commit rules in `AGENTS.md`, `CLAUDE.md`
  or `CONTRIBUTING`: prefix, tense, language. Subject under ~72 characters; a body only for the why.
- A failing hook is a problem to fix, not to skip: never `--no-verify`. Do not push or amend.
- Say the commit's short hash and subject, and what was left uncommitted.

$ARGUMENTS
