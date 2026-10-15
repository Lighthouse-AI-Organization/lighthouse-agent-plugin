---
name: connect-sources
description: Connect a data source (Postgres, Snowflake, BigQuery, SQL Server, HTTP connector, Mixpanel), a Slack or email notification destination, or an integration action source to Lighthouse, or reconnect one with new credentials, through a secure Lighthouse link so credentials never pass through the chat. Use when the user wants to connect, add, reconnect or fix the credentials of a source in Lighthouse.
---

# Connect sources

Credentials never go through this conversation. The begin tools return a Lighthouse link; the user opens it while signed in, enters the details there and saves. You then check how it ended.

If a tool named here is missing, the user's key or role does not allow it. Say which one and that it can be changed in Lighthouse under Settings, API & Agents.

## Steps

1. **Check what exists.** `list_data_sources` or `list_communication_sources`, so you do not create a duplicate. To reconnect, find the id with `get_data_source`.
2. **Start the setup.**
    - Data source: `begin_data_source_setup` with the dataSourceType (POSTGRES, SNOWFLAKE, BIGQUERY, MSSQL, HTTP_CONNECTOR, MIXPANEL) to create, or the dataSourceId to replace credentials or connection settings.
    - Slack or email notifications: `begin_communication_source_setup` with type SLACK (the user approves Lighthouse on Slack's own page) or EMAIL (an organization can have only one).
    - A place Lighthouse sends actions to, such as an HTTP endpoint: `begin_integration_action_source_setup`, with the integrationActionSourceId to reconnect one.
3. **Hand over the link.** Give the user the url and say: open it while signed in to Lighthouse in the same organization, fill in the details, save. It works for 20 minutes and only for them. For a warehouse, suggest a read-only database user or role.
4. **Check the result** when the user says they are done: `get_setup_status` with the setupId.
    - completed: entityId is the new or updated source.
    - pending or in_progress: they have not saved yet, or the save is running. Wait and check again.
    - expired: start again for a new link.
    - unknown: the save may or may not have happened. Check `list_data_sources` or `list_communication_sources` (or Lighthouse itself, for an integration action source) before starting another setup.
5. **Tidy up.** Rename or describe a source with `update_data_source` or `update_integration_action_source`. These never change credentials or connection settings.
6. **Next step.** For a new data source, offer to explore it (explore-data-and-model-datasets skill).

## Safety rules

- Never ask for a password, key, token or connection string in the chat, and never put one in a tool argument.
- If the user pastes a secret anyway, do not use it. Tell them to use the link instead and to rotate that secret, since it is now in the chat history.
- Calling a begin tool again for the same target returns the same link while it is waiting; do not spam new setups.

## Report back

- What was connected or reconnected, its name and id, from `get_setup_status`.
- Anything left for the user (a link still waiting, an expired link, a Slack approval).
- Suggested next step.
