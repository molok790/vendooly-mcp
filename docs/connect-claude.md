# Connect Vendooly to Claude

Vendooly is a remote MCP server: in Claude it is added as a **custom connector**. Nothing to install.

## Before you start

1. A Vendooly account: https://vendooly.com/signup.html (14-day free trial).
2. At least one seller account authorized in the Vendooly app (Marketplaces → choose the region → authorize on Seller Central). For Amazon Ads press **Connect Ads** as well: it is a separate authorization.
3. A Claude plan that supports custom connectors. If you are in a Claude organization, your admin may need to enable or add the connector.

## Steps (claude.ai and Claude Desktop)

1. Open **Settings → Connectors**.
2. Click **Add custom connector**.
3. Fill in:
   - **Name**: `Vendooly`
   - **URL**: `https://mcp.vendooly.com/mcp`
4. Confirm. Claude opens the Vendooly consent page: sign in with your Vendooly account and authorize.
5. In a chat, make sure the Vendooly connector is enabled (tools menu of the chat).

## Claude Code

```bash
claude mcp add --transport http vendooly https://mcp.vendooly.com/mcp
```

Then run `/mcp` in Claude Code and complete the authorization in the browser.

## Check that it works

> Now that you are connected to Vendooly, tell me which seller accounts and marketplaces you can see and what you can help me with.

If Claude lists your seller accounts, you are ready. If it says it cannot see Amazon Ads, go back to Marketplaces in the Vendooly app and check that both Seller and Ads are connected.

## Permissions

Claude asks your permission before calling a tool that changes something ("Allow once" / "Always allow"). On top of that, Vendooly instructs the assistant to show a before → after preview and wait for your OK before any write. We recommend keeping write tools on "Allow once". See [security.md](security.md).

## Troubleshooting

- **Connector added but no tools**: remove and re-add the connector, then authorize again.
- **"Ads not connected" but the app says they are**: Seller and Ads are two separate authorizations; reconnect Ads from the app.
- Anything else: hello@vendooly.com, or ask Claude to send feedback to the Vendooly team (`send_feedback`).
