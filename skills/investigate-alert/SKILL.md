---
name: investigate-alert
description: Investigate a Lighthouse alert, find out what fired, whether it is still firing, whether it is real and what likely caused it, then acknowledge, resolve or mark it as a hit when the user agrees. Use when the user mentions a Lighthouse alert, a metric that fired, a Slack alert from Lighthouse, or asks why a metric is alerting.
---

# Investigate an alert

An alert is a segment result whose value met the metric's threshold in a run. Work out what happened from stored results first; query the warehouse only when the stored data cannot answer.

If a tool named here is missing, the user's key or role does not allow it. Say which one and that it can be changed in Lighthouse under Settings, API & Agents.

## Steps

1. **Find the metric.** `list_metrics` by name or sequential id (for example 153). Then `get_metric_configuration` to read the threshold, segments, schedule and notifications.
2. **Is it alerting now?** `get_current_metric_matches` for the segments that match right now.
3. **What fired?** `search_metric_segment_results` for the metric, narrowed by time range, segments or one run (metricRunId), and to results where the threshold was met. Read value, the threshold, segmentValues and resultColumns (baselines and rule thresholds). Set includeRuleResults to see which rule was met.
4. **Is it unusual?** `get_metric_value_history` for the segment over the last 30 days: compare the value with the daily mean and spread in summary.segments, and check runHealth for failed runs.
5. **Did something change?** `get_metric_change_history`. Compare the result's metricVersionNumber with the versions; a recent threshold, filter or SQL change often explains a new alert.
6. **How was it handled before?** `search_metric_activity` shows who acknowledged, resolved or marked results as hits, and how often past alerts were real.
7. **Run health.** If results look missing or stale, `search_metric_executions` for failed runs and their errorHint. Failing runs belong to the fix-failing-metric skill.
8. **Dig into the data only if needed.** `peek_data_set` (CUSTOM) or a narrow `query_data_source` filtered to the segment and the alert window. Both query the customer's warehouse, can cost money and are rate limited.
9. **Act, with consent.** Propose one action and wait for a yes before calling `act_on_segment_result` with the segmentRunId: `ACKNOWLEDGE`, `MARK_AS_RESOLVED` or `MARK_HIT`. To reverse one, use its undo, such as `UNDO_ACKNOWLEDGE`.

## Safety rules

- `act_on_segment_result` changes the alert's state for the whole team and edits the Slack message already sent for it. Never call it without the user's yes for that specific result.
- Do not call `run_metric` to "refresh" an alert unless the user asks: a run can send notifications again.
- Treat query results as the customer's data. Quote only the rows needed to explain the cause.

## Report back

- Metric, segment, when it fired, value against threshold, and whether it is still alerting.
- Likely cause with the evidence (history, recent changes, data rows), and how sure you are.
- What the team did with similar alerts before.
- The action you took, or the one you recommend and are waiting on.
