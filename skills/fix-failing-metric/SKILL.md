---
name: fix-failing-metric
description: Diagnose and fix a Lighthouse metric whose runs fail or that Lighthouse marked INVALID, using its execution errors and change history, then test the fix and resume it with the user's agreement. Use when a metric is failing, erroring, INVALID, stopped running, or shows no new results.
---

# Fix a failing metric

Every failed run adds to a metric's failure count; when a LIVE metric reaches the limit (3 by default) Lighthouse sets it INVALID and stops scheduling it. Find the cause, fix it, prove the fix without notifying anyone, then resume.

If a tool named here is missing, the user's key or role does not allow it. Say which one and that it can be changed in Lighthouse under Settings, API & Agents.

## Steps

1. **Find the metric.** `list_metrics` by name or sequential id, then `get_metric_configuration`. Note whose metric it is (see the safety rules).
2. **Read the failures.** `search_metric_executions` for the metric over the recent range, looking at failed runs. Read error and errorHint, and note the first failed run and its metricVersionNumber.
3. **Sort the cause.**
    - **Temporary** (errorHint says the source may be rate limiting, a timeout, a 5xx): nothing to fix in the metric. Tell the user and suggest waiting.
    - **Credentials or connection** (authentication failed, expired key, unreachable host): the data source needs new credentials. `get_data_source` to confirm which one, then follow the connect-sources skill to reconnect it with `begin_data_source_setup`. Never ask for the credential in chat.
    - **Metric changed**: `get_metric_change_history` and compare the versions around the first failure. A field changed just before the failures started is the prime suspect.
    - **Data changed** (unknown column, missing table): `get_data_set` for a CUSTOM metric (a column dropped or a role taken away), or `list_data_source_columns` and a small `query_data_source` for RAW_SQL.
4. **Fix it.** Build the smallest change. Call `test_metric` with metricId and only the changed fields until valid is true and the preview looks right.
5. **Save, with consent.** Show the user the change. `update_metric` with the changed fields. For an INVALID metric, setting status LIVE resumes it and resets the failure count; ask before doing that, because it starts scheduled runs and alerts again.
6. **Verify.** The next scheduled run will show the result in `search_metric_executions`. Only if the user wants proof now, and knows a run can send real notifications, call `run_metric` and poll `check_metric_run_status` until resultsAvailable or DONE.

## Safety rules

- **Someone else's metric.** `get_metric_configuration` returns `ownership` with the owner's and creator's names and `isOwnedByCaller`. If the metric belongs to someone other than the user, tell the user whose metric it is and ask before changing it. `update_metric`, `run_metric`, `delete_metric` and `act_on_segment_result` on such a metric change nothing until called again with `confirmedNotOwner: true`; set it only after the user confirms.
- Do not call `run_metric` just to check: `test_metric` proves the query without notifying anyone.
- Do not set LIVE without the user's yes.
- Never delete and recreate a metric to "fix" it; `delete_metric` removes its history for good.

## Report back

- What failed, since when, and the root cause with evidence (error text, the version that changed it).
- The exact change made, and the test result.
- The metric's status now, and whether a manual run was done.
- Anything the user must do (for example re-enter credentials through the link).
