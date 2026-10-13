---
name: pr-description
description: Write a pull request's title and description from the branch's commits and diff — what changes for the user, why, how it was checked, what to look at closely. Use when asked to write, draft or update a PR or MR description, or to open a pull request.
---

# Describe a pull request

1. Find the range: the base branch (`main` unless the user or the repo says otherwise) and
   `git log --oneline <base>..HEAD`, `git diff --stat <base>...HEAD`, then the diff itself for
   anything the commit messages do not explain.
2. A template wins: `.github/pull_request_template.md` (or the GitLab equivalent) → fill it in,
   keep its headings.
3. Otherwise write:
   - **Title** — what the change does, in the style of the repo's merged PRs (`gh pr list --state merged
     --limit 10` shows it), under ~70 characters.
   - **What and why** — the change from the user's or caller's side first, then the reason.
     A few bullets, not a file-by-file tour.
   - **How it was checked** — only commands that were actually run in this session or by CI, with
     their outcome. Never write "tests pass" without having seen them pass; say what was not checked.
   - **Review notes** — risky spots, migrations, config or API changes, follow-ups left out on purpose.
   - Linked issues (`Fixes #123`) when the commits or the user name them.
4. Unrelated commits in the range → list them under their own heading instead of hiding them.
5. Show the text to the user. Open or edit the PR (`gh pr create` / `gh pr edit`, `glab mr create`)
   only when they asked for that, and give them its link.
