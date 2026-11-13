---
name: testops-mcp
description: Work with Allure TestOps through the testops MCP server — find test cases and read their steps, search with AQL, run a selection in a launch, record results, read launch statistics and the history of a case. Use when a task mentions Allure TestOps, test cases, test plans, launches or test results.
---

# Work with Allure TestOps

The `testops` server has 30 tools: 17 read and 13 write. Every answer is a JSON string — `{"ok": true, ...}`
on success, `{"ok": false, "error": "HTTP 404 on GET /api/testcase/1: test case not found"}` on failure.
The tools never raise, so read the reason out of the `error` field instead of treating the call as broken.

## 1. Do not invent identifiers

`project_id` comes from `list_projects`; if `TESTOPS_PROJECT_ID` is set on the server, the tools fall back
to it. `status_id`, `workflow_id` and `test_layer_id` come from the reference tools only:

- `list_statuses(workflow_id=...)` → `{id, name, color}`;
- `list_reference("workflow")`, `list_reference("testlayer")`,
  `list_reference("custom_field", project_id=...)`.

An invented `status_id` answers 400 — or, worse, creates a case in the wrong status.

## 2. Read before you change

| The question | The tool |
|---|---|
| which projects exist | `list_projects(search=...)` |
| the cases of a project, one row each | `list_test_cases(project_id=...)` |
| find cases by name or by AQL | `search_test_cases(name=...)`, `search_test_cases(rql=...)` |
| check an AQL query before using it | `validate_aql(rql=...)` → `{valid, count}` |
| one case in full | `get_test_case(id)` |
| the steps of a case | `get_test_case_steps(id)` |
| where a case has run | `get_test_case_history(test_case_id=...)` |
| test plans | `list_test_plans(project_id=...)`, `get_test_plan(id)` |
| launches of a project | `list_launches(project_id=...)` |
| results of a launch | `list_test_results(launch_id=...)`, `get_test_result(id)`, `get_launch_statistic(launch_id)` |

`get_test_case` does **not** carry steps: the answer says `"steps_not_included": true` and the steps come
from `get_test_case_steps`. `list_test_cases` rows carry neither descriptions nor steps.

## 3. Recipes

### AQL: what the stand accepts

Grammar is the one in [docs.qameta.io/reference/aql](https://docs.qameta.io/reference/aql), checked
against a live stand. Substring matching is `~=` only, and the field names are not the DTO's:

- `name ~= "login"` (substring), `name = "Login smoke"` (exact), `name != "x"`;
- `tag = "smoke"`, `tag in ["smoke", "regress"]`, `not tag = "smoke"` — not `tag.name`;
- `status = "Active"`, `workflow = "Default Manual"`, `layer = "API"` — not `status.name`/`testLayer`;
- `automation = true`, `flaky = false`, `muted = true` — not `automated`;
- `cf["Severity"] = "high"`, `cfv in ["high"]`, `role["Owner"] = "user1"`;
- `id >= 100`, `createdDate > 1672531200000` — dates are epoch milliseconds, unquoted;
- conditions join with `and` / `or` (`and` binds tighter), grouping is parentheses.

A wrong query answers `400 invalid.aql` and `validate_aql` says `{valid: false}` — so validate first,
search second.

**Find a case and read it.** `search_test_cases(name="login")` turns the name into AQL `name ~= "login"` on
the server — there is no client-side filtering. For a compound condition, `validate_aql(rql='tag = "smoke"')`
first and pass the same `rql` to `search_test_cases` when it is valid. Then `get_test_case(id)`, then
`get_test_case_steps(id)`.

**Find out where a case ran.** `get_test_case_history(test_case_id=..., project_id=...)` → rows of
`{testResultId, launchId, launchName, status, start, duration}`. Narrow it with `launch_id=...` or
`search=...`; read one row in full with `get_test_result(testResultId)`.

**Run a selection in a new launch.** Take the ids from `search_test_cases` — for `run_test_cases` they are
tree **leaf** ids — and call `run_test_cases(launch_name="Regression 2026-11-09", ids="1001,1002")`, which
answers the `launchId`. `leafs_include="[1001,1002]"` or a saved `filter_id=...` work instead of `ids`;
passing `ids` and `leafs_include` together is an error. The launch is created with manual results, one per
case; check it with `list_test_results(launch_id=...)` or `get_launch_statistic(launch_id=...)`.

**Record a result in an existing launch.** `create_launch(name="Manual run", project_id=...)` → `id`, then
`add_test_result(launch_id=..., name="Login smoke", status="failed", message="...", duration=1300, test_case_id=...)`.
`status` is one of `failed | broken | passed | skipped | unknown | in_progress` — anything else is refused
before the request. `duration`, `start` and `stop` are milliseconds. `close_launch(launch_id=...)` makes the
results read-only; `reopen_launch(launch_id=...)` undoes that.

**Create a case with steps.** `create_test_case(name="Login smoke", project_id=..., status_id=..., tags_json='[{"id": 3, "name": "smoke"}]')`
→ `id`, then `add_test_case_step(test_case_id=..., body="Open the form")` — exactly one of `body`,
`attachment_id`, `shared_step_id`; `parent_step_id` nests a step. Verify with `get_test_case_steps(id)`.

**Clean up after yourself.** `delete_test_case(id, force=True)` soft-deletes (202) and puts the case in the
bin, with no restore tool; `delete_launch(launch_id)` removes the launch with its results;
`delete_test_plan(test_plan_id)` leaves the cases alone.

**A test plan.** `create_test_plan(name="Smoke", project_id=..., base_rql='tag = "smoke"', tree_selection_json='{"leafsInclude": [1001]}')`;
`update_test_plan(id, name=...)` builds the body from the fields you pass only, and `clear=true` empties a
string field.

## 4. How to pass arguments

- **Structured values go in as JSON strings**: `tags_json`, `links_json`, `members_json`,
  `custom_fields_json`, `scenario_json`, `tree_selection_json`. The exact shape is in the tool's docstring.
  `tags_json` → `[{"id": 3, "name": "smoke"}]` (the id is required), `members_json` → the role as an object
  `{"id": 1, "name": "MEMBER"}`, `custom_fields_json` → `[{"customField": {"id": 12, "name": "Severity"}, "name": "high"}]`.
- **Paging**: `page` starts at 0, `size` defaults to 50 and stops at 100; the answer carries
  `totalElements`, `totalPages` and `number`, so a partial selection is visible.
- **Sorting**: `sort="createdDate,DESC"`.
- **`clear`** in `update_test_case` and `update_test_plan`: without it an empty string means "leave the field
  alone", with it "empty the field".

## 5. What the tools cannot do

- There is no tree, no shared steps, no defects, no attachments and no result import — and no generic
  "any path under /api" tool to work around it. If a tool is missing, the task does not go through this MCP.
- `run_test_cases` has no `rql`: a selection is `ids`, `leafs_include` or `filter_id`.
- Attachments come back as ids, not contents; the HTML duplicates of text fields are stripped, so do not
  look for `*Html` keys.

## 6. When the answer is not what you expected

- `HTTP 401` or `403` — authentication: the server sends `Authorization: Api-Token <token>` and the stand may
  want `Bearer`. That is a configuration question, not a tool problem.
- `HTTP 400` mentioning rql — check the AQL with `validate_aql` and search again.
- `{"ok": true, "accepted": true}` — the operation was accepted asynchronously (202 or 204). If you need an
  id that is not in the answer, read the state back with `list_*` or `get_*`.
- An empty `content` — check `project_id`, and pass `deleted=true` to see cases that are in the bin.

## 7. Safety

- Call `delete_*` only when the user asked for a deletion.
- `force=true` only for what this same run created.
- If the write tools are missing from the list, the server runs with `TESTOPS_READ_ONLY=true`. That is
  deliberate: say writing is off instead of looking for a way around it.

## 8. Is the server alive

The cheapest call is `list_projects(search="")`: `{"ok": true, "content": [...]}` means the token and the
stand address are right. `{"ok": false, "error": "TESTOPS_TOKEN is not set"}` means the server did not start
at all — a client configuration problem.
