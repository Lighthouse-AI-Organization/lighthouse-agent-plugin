---
name: build-metric
description: Build a new Lighthouse metric (a scheduled check on the user's data that alerts in Slack or email when a threshold is met) from a plain-language goal, test it, backtest it and take it live. Use when the user asks to monitor, track, watch or alert on something in their data with Lighthouse.
---

# Build a metric

A metric runs a query on a data source on a schedule, compares the result with a threshold and notifies people when it is met. Build it as a DRAFT, prove it works, then go live only when the user says so.

If a tool named here is missing, the user's key or role does not allow it. Say which one and that it can be changed in Lighthouse under Settings, API & Agents.

## Steps

1. **Pin down the goal.** What number, per what (segments such as country or customer), how often, what counts as bad, and who should hear about it. Ask only for what you cannot find.
2. **Find the data.**
    - `list_data_sources` and `list_data_sets` for what exists; `get_data_set` for column ids and which columns can be a metric, filter, segment or date.
    - Prefer a CUSTOM metric on a data set: it gets trend and threshold options and can be backtested. Use RAW_SQL only when no data set fits and one should not be made.
    - If no data set fits, follow the explore-data-and-model-datasets skill (`list_data_source_tables`, `list_data_source_columns`, `query_data_source`, `create_data_set`).
3. **Start from an example.** Call `search_metric_configuration_examples` with words from the goal (for example "count", "segments", "trend max", "calculation window", "slack"), then peek at up to 5 example ids for the full JSON. Copy the closest shape; do not invent fields.
4. **Assemble the configuration** with real ids only:
    - data source and data set ids from step 2;
    - a schedule (every metric needs one);
    - notifications: `list_communication_sources`, then `get_communication_source` for Slack channel or user ids. Email recipients must be members of the organization;
    - KPI tags if the user wants them (`list_kpis`);
    - who may edit and view it. Ask the user; do not assume. For editing: everyone (`editAccessLevel` `EVERYONE`, "Everyone" in the web app) or only them (`ADMINS`, shown as "Owner"). For viewing, the same choice with `viewAccessLevel`. Pass the choice in the configuration. To share with specific people or groups, ask the user to set it in the Lighthouse web app after the metric is saved.
5. **Test.** Call `test_metric` with the full configuration. Fix every error and repeat until valid is true. Show the user the preview rows and check they look like the number they meant.
6. **Save as a draft.** `create_metric` with status DRAFT. A draft never runs on its schedule and sends nothing.
7. **Backtest.** For a CUSTOM metric with a calculation function, call `preview_metric_history` with the new metric id. Report how often the threshold would have been met per day. If it would fire too often or never, adjust the threshold with `test_metric` and `preview_metric_history` (pass metricId and only the changed fields), then save with `update_metric`. See the tune-alert-noise skill for more.
8. **Go live, with consent.** Summarize what will run, who will be notified and who can edit and view it, and ask. Only after a clear yes, set status LIVE with `update_metric`. LIVE counts toward the plan's live metric limit.

## Safety rules

- Ask before `run_metric`: a run can send the metric's real notifications and actions, like a scheduled run. `test_metric` and `preview_metric_history` never notify anyone.
- Never call `send_test_notification` unless the user asked for a test message. It sends a real message to real people; call it once.
- `test_metric` and `preview_metric_history` query the customer's warehouse, which can cost money and is rate limited. Do not loop on them; change something between calls.
- Do not set a metric LIVE or PAUSED without the user's yes.

## Report back

- Metric name and id, status, data source or data set, schedule.
- The threshold in plain words, the segments, who is notified and where, and who can edit and view it.
- The test preview and the backtest result (hits per day, days with hits).
- What is still open (for example "still a DRAFT, say the word to go live").
