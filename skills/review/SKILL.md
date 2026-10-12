---
name: review
description: Review the current change for bugs before it is committed — read the diff, check it against the checklist, report findings ranked by severity. Use when asked to review, check or look over a change, a diff or a branch.
---

# Review a change

1. Get the change: `git diff` for unstaged work, `git diff --staged` for what is staged,
   `git diff main...HEAD` for a branch. If the user named a file or a commit, review that.
2. Read every changed hunk, then open the surrounding code a hunk depends on — a call site,
   the type it returns, the test that covers it. A diff alone hides most bugs.
3. Go through [checklist.md](checklist.md) for each hunk.
4. Report findings, most severe first. For each: `path:line`, what is wrong, a concrete input or
   state that triggers it, and the fix in a sentence. Say plainly when you found nothing.

Do not edit files while reviewing unless the user asks for fixes.
