# Lighthouse agent plugin

Skills and an MCP connection that let AI agents (Claude, Cursor, Codex and others) work with [Lighthouse](https://lighthouseai.co): build and test metrics, investigate alerts, fix failing metrics, explore your data, tune alert noise, connect sources and manage members.

Installing the plugin connects the Lighthouse MCP server at `https://api.lighthouseai.co/public-api/mcp`. You sign in with your Lighthouse account, and the agent can only do what your own Lighthouse permissions allow.

## Install

### Claude Code

```
/plugin marketplace add Lighthouse-AI-Organization/lighthouse-agent-plugin
/plugin install lighthouse@lighthouse
```

Restart the session, then sign in to Lighthouse when the agent first calls a tool (or run `/mcp`).

### Claude (claude.ai and Claude Desktop)

1. Add the Lighthouse connector: Settings > Connectors > Add custom connector, URL `https://api.lighthouseai.co/public-api/mcp`.
2. For the skills, zip any folder under `skills/` and upload it in Settings > Capabilities > Skills.

### Cursor, VS Code and other agents

Add the MCP server from Lighthouse under Settings > API & Agents > Connect an agent, then copy the `skills/` folder into your agent's skills or rules folder.

If you already added the Lighthouse connector by hand, installing the plugin adds a second Lighthouse server. Keep one of them.

## Skills

| Skill                              | Use it to                                                           |
| ---------------------------------- | ------------------------------------------------------------------- |
| `build-metric`                     | Create a metric: find examples, test it, save as draft, go live     |
| `investigate-alert`                | Understand why a metric alerted and act on the result               |
| `fix-failing-metric`               | Find why a metric fails, using its run errors and change history    |
| `explore-data-and-model-datasets`  | Browse tables and columns, preview data, create or update datasets  |
| `tune-alert-noise`                 | Backtest thresholds and reduce noisy alerts                         |
| `connect-sources`                  | Connect data sources and Slack or email, without pasting secrets    |
| `admin-members-and-groups`         | For admins: members, roles, invitations and business groups         |

Every skill asks before anything that sends a message, changes access, or makes a metric live.

## Support

Questions or problems: [lighthouseai.co](https://lighthouseai.co).
