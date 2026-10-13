---
name: commit
description: Commit the change in the working tree with a message in this repository's own style — what is staged or what the user names, one concern per commit, nothing unrelated swept in. Use when asked to commit, make a commit, or save the work to git.
---

# Commit a change

1. See what there is: `git status` and `git diff` (unstaged), `git diff --staged` (staged).
   - Something is staged already → commit that, unless the user named other files.
   - Nothing staged → stage what the user asked for, by path. Never `git add -A` or `git add .`
     when there are files the user did not mention: list them and ask.
   - Two unrelated changes in the tree → offer two commits rather than one.
2. Learn the house style before writing a word: `git log --oneline -15` and one full message
   (`git log -1`). Match its prefix (`fix:`, `area: …`, a ticket id), tense, language and length.
   A `CONTRIBUTING`, `AGENTS.md` or `CLAUDE.md` with commit rules outranks the log.
3. Write the message:
   - Subject: what the change does, under ~72 characters, no trailing period.
   - Body, when the subject cannot carry it: why, and what someone reading `git blame` needs —
     not a list of the files touched.
4. Commit with the message passed as a file or heredoc, not squeezed into one `-m` line.
   A hook that fails is a finding: show its output and fix the cause; never `--no-verify`.
5. Report the commit's short hash and subject, and what was left uncommitted.

Do not push, amend an existing commit or rewrite history unless the user asks for exactly that.
Do not commit secrets: a `.env`, a key or a token in the diff → stop and say so.
