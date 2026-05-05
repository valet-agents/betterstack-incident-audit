# Better Stack Incident Audit

When an incident fires, it captures the timeline, postmortem owner, and the evidence to preserve — before anyone forgets the details.

## Prerequisites
- A [Better Stack](https://betterstack.com) account with an API token and at least one monitor configured
- A Slack workspace where you can install the agent's bot and invite it to one or more channels

<table>
  <tr>
    <td><strong>CHANNELS</strong></td>
    <td><code>slack</code> · <code>heartbeat</code> — every 2m</td>
  </tr>
  <tr>
    <td><strong>CONNECTORS</strong></td>
    <td><code>betterstack-mcp</code></td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <br />
      <a href="https://valet.dev/deploy?from=github.com/valet-agents/betterstack-incident-audit">
        <img src="https://raw.githubusercontent.com/valet-agents/betterstack-incident-audit/main/.github/deploy-button.svg" alt="Deploy Agent →" height="40" />
      </a>
      <br /><br />
    </td>
  </tr>
</table>
