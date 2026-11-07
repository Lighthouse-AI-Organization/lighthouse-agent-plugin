---
name: explore-data-and-model-datasets
description: Explore the tables and columns in a Lighthouse data source, run narrow read-only queries to understand the data, and create or adjust Lighthouse data sets that metrics are built on. Use when the user asks what data Lighthouse can see, what could be monitored, or to model a table as a data set.
---

# Explore data and model data sets

A data source is a connection to a warehouse or app. A data set is one of its tables, with roles for each column (metric, filter, segment, date). CUSTOM metrics are built on data sets.

If a tool named here is missing, the user's key or role does not allow it. Say which one and that it can be changed in Lighthouse under Settings, API & Agents.

## Steps

1. **See what exists.** `list_data_sources`, `get_data_source`, `list_data_sets` and `get_data_set`. Reuse a data set that already covers the table.
2. **Browse the schema (free).** `list_data_source_tables` and `list_data_source_columns` run no warehouse query. Search by name; page through large schemas.
3. **Look at real values (costs a query).**
    - For an existing data set, `peek_data_set` with its column ids and a date range.
    - Otherwise `query_data_source` with one narrow SELECT: only the columns you need, a date filter, aggregates or a small LIMIT. Copy table and column names from step 2.
    - Both work on warehouse data sources (Postgres, Snowflake, BigQuery) only.
    - Each query runs on the customer's warehouse, can cost money, and shares a per-minute limit (10 by default) with `peek_data_set` and `preview_metric_history`. Plan queries; do not fire them in a loop.
4. **Propose a data set.** Name the table, the default date column (must be a date or timestamp), which number columns can be metrics, and which text columns are segments. Show this to the user.
5. **Create it, with consent.** `create_data_set` from one table. Leave out columns to take every column with default roles, or list the columns you want. Data sets defined by custom SQL are not supported.
6. **Adjust it.** `get_data_set` first, then `update_data_set` with only what changes.
7. **Next step.** Offer to build a metric on it (build-metric skill).

## Safety rules

- Query results are the customer's data. Show only what answers the question; never copy rows anywhere the user did not ask for.
- `query_data_source` only accepts one read-only SELECT or WITH. If it refuses a query, fix the SQL rather than working around the check.
- Before `update_data_set` removes a column from metrics or takes a role away, warn the user: metrics built on the data set change with it and can start failing. Ask for a yes.
- Recommend a read-only database user for the data source if the user is unsure what it can reach.
- Deleting a data set or data source is for organization admins (admin-members-and-groups skill). Never delete one to start over; adjust it with `update_data_set`.

## Report back

- What the data source contains that matters for the user's goal (tables, key columns, grain, date range).
- Findings from queries, as short numbers or a few rows, with the SQL you ran.
- Data sets created or changed: name, id, date column, metric and segment columns.
- Metrics this suggests, if the user wants them.
