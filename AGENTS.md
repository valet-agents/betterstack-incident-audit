This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

- **betterstack-mcp**: The Better Stack MCP server, authenticated with an API token. The agent uses it to list incidents, pull timelines, fetch severity / status / owner, and read the metadata that becomes the audit trail. Add it from the catalog at the org level so other Better Stack-powered agents can share it.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts audit-ready incident timelines to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **heartbeat** (heartbeat): Fires every 2 minutes (`every: 2m`). Polls Better Stack for new incidents and queues resolution replies for any tracked incident that just closed. Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow.

### Secrets

- **BETTERSTACK_API_TOKEN** (on the `betterstack-mcp` connector): Better Stack API token with read access to incidents, monitors, and policies. Add write access (`incidents:write`) if you plan to use the confirmed-ack/resolve flow from Slack. Create one at betterstack.com → User Settings → API tokens.

### External Setup

1. **Create a Better Stack API token**: betterstack.com → User Settings → API tokens → Create. Give it read access to incidents, monitors, and policies. Optionally add write access if you want Slack users to be able to ack or resolve from the thread.
2. **Confirm at least one monitor exists**: the agent only posts timelines for incidents tied to a monitor or policy. If your workspace has no monitors yet, configure one in Better Stack first.
3. **Invite the Slack bot**: Invite the agent's bot to whichever channel(s) you want incident timelines to land in. The agent posts to every channel it's a member of — invite it to one focused on-call channel, or several. If the bot has not been invited anywhere, timelines go as a DM to the workspace install user with a one-line nudge.
4. **Smoke-test**: trigger a test incident in Better Stack (or wait for a real one), or @mention the bot with a question like *"any open incidents?"* — that exercises the Slack + Better Stack path without waiting for the next heartbeat.

## Customizing

- **Change the heartbeat interval**: edit `every` on the `heartbeat` channel in `valet.yaml`, then redeploy. The default is 2 minutes — incidents are time-sensitive, so going much higher trades responsiveness for fewer pollings.
- **Set a default postmortem owner**: set the `DEFAULT_POSTMORTEM_OWNER` env var on the agent (a Slack `@user` handle, an email, or a name). The SOUL falls back to it when Better Stack hasn't assigned an owner yet, and to `unassigned` if neither is set.
- **Tune which evidence pointers get included**: the SOUL **Phase 2** list (incident URL, dashboard URL, log query, error count at fire) is the default. Add or remove pointers there if your team's audit process expects more or fewer artifacts.
- **Control where timelines post**: invite or remove the bot from channels in Slack — that's the only signal the agent uses. There is no channel name in the configuration.
- **Tune the de-dup**: the agent tracks the watermark + open-incident map in `MEMORY.md`. The watermark gates new posts; the map gates the resolution reply. Don't edit it by hand unless you want to replay or skip a backlog.
