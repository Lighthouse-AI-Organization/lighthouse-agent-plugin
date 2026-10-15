---
name: tune-alert-noise
description: Make a noisy or silent Lighthouse metric alert at the right rate by measuring its history, backtesting candidate thresholds or filters, and applying the best one with the user's agreement. Use when the user says a metric alerts too often, too rarely, is noisy, has false positives, or asks to tune or adjust a threshold.
---

# Tune alert noise

Measure first, then backtest options side by side, then change one thing. A good threshold fires on the cases the team treats as real and stays quiet otherwise.

If a tool named here is missing, the user's key or role does not allow it. Say which one and that it can be changed in Lighthouse under Settings, API & Agents.

## Steps

1. **Read the metric.** `list_metrics`, then `get_metric_configuration` for the threshold, filters, segments, trend settings and schedule, and whose metric it is (see the safety rules).
2. **Measure the noise.** `get_metric_value_history` over 30 to 90 days. Use summary.segments: threshold-met runs and days, mean, standard deviation and coefficient of variation per segment. Note which segments cause most of the hits.
3. **Check what the team thought.** `search_metric_activity`: many acknowledged or resolved results and few marked as hits means false alarms; hits marked real are alerts to keep.
4. **Check past attempts.** `get_metric_change_history` to see whether the threshold was already moved and what happened.
5. **Look at a few alerts.** `search_metric_segment_results` with the threshold met and includeRuleResults, to see which rule fires and by how much.
6. **Backtest candidates.** For each option (a higher threshold, a trend comparison, a minimum volume filter, excluding a segment), call `preview_metric_history` with metricId and only the changed fields. Use `search_metric_configuration_examples` for the right JSON shape. Each backtest queries the warehouse twice, can cost money and is rate limited, so try two or three options, not ten. Only CUSTOM metrics with a calculation function can be backtested.
7. **Recommend one.** A small table: option, hits in total, days with hits, which past real hits it keeps.
8. **Apply, with consent.** `test_metric` with the chosen change, then `update_metric`. To trial a change without touching the original, `duplicate_metric` makes a DRAFT copy that sends nothing until it is made LIVE.

## Safety rules

- **Someone else's metric.** `get_metric_configuration` returns the metric's `ownerId`. If it is not the user's own metric, tell the user whose metric it is (the oldest entry in `get_metric_change_history` names who created it) and ask before changing, running or deleting it.
- Do not change a LIVE metric without the user's yes on the exact change.
- `send_test_notification` sends a real message to the metric's channel or recipients. Call it only when the user asks to see the new alert, and only once.
- Do not use `run_metric` to test a threshold: it can send real alerts. Use `preview_metric_history` or `test_metric`.

## Report back

- Current noise: hits per day and the segments behind them, and how the team treated them.
- The options you backtested with their numbers.
- The change applied (or recommended), and the expected alert rate.
- Any DRAFT copy you created, with its id.
