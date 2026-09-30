# Connect Vendooly to ChatGPT

Vendooly is a remote MCP server: in ChatGPT it is added as a **connector** (app). Nothing to install.

## Before you start

1. A Vendooly account: https://vendooly.com/signup.html (14-day free trial).
2. At least one seller account authorized in the Vendooly app (Marketplaces → choose the region → authorize on Seller Central). For Amazon Ads press **Connect Ads** as well: it is a separate authorization.
3. A ChatGPT plan with custom connectors. Depending on plan and workspace, you may need to enable **Developer mode** (Settings → Connectors → Advanced settings) or ask your workspace admin.

## Steps

1. Open **Settings → Connectors** (called **Apps** in some versions).
2. Click **Create** (at the bottom of the page).
3. Fill in:
   - **Name**: `Vendooly`
   - **MCP server URL**: `https://mcp.vendooly.com/mcp`
   - **Authentication**: `OAuth`
4. Confirm. ChatGPT opens the Vendooly consent page: sign in with your Vendooly account and authorize.
5. In a new chat, select the Vendooly connector (the **+** / tools menu of the composer).

## Check that it works

> Now that you are connected to Vendooly, tell me which seller accounts and marketplaces you can see and what you can help me with.

## Permissions

Write tools are annotated as non-read-only, so ChatGPT asks for confirmation before running them. Vendooly additionally instructs the assistant to show a before → after preview and wait for your explicit OK. See [security.md](security.md).

## OpenAI Agents SDK and other frameworks

Any framework with MCP support (OpenAI Agents SDK, LangChain, Pydantic AI, CrewAI, the official MCP SDKs) can use the same URL with streamable HTTP and OAuth. A tested Python example with the official `mcp` SDK is in the Italian guide: https://vendooly.com/guida.html

## Troubleshooting

- **No "Create" button**: your plan or workspace does not allow custom connectors yet, or Developer mode is off.
- **"Ads not connected" but the app says they are**: Seller and Ads are two separate authorizations; reconnect Ads from the app.
- Anything else: hello@vendooly.com
