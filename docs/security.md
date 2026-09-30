# Security model

Vendooly sits between your AI assistant and Amazon. The assistant can only call a defined set of tools; authorization, Amazon tokens, tenant boundaries and write approvals stay outside the model.

## Authorization

- **Client to Vendooly**: OAuth 2.1 with Dynamic Client Registration and PKCE (S256). Your MCP client registers itself, opens the Vendooly consent page in the browser and receives a token. No API key to copy, no password in the chat.
- **Vendooly to Amazon**: Amazon's official OAuth for the Selling Partner API (Seller Central, Vendor Central) and, separately, for the Amazon Ads API. You grant it from the Vendooly app and can revoke it at any time from Amazon or from Vendooly.
- Amazon refresh tokens are **encrypted at rest** on the server. They are never inserted into prompts, never returned to the assistant and never sent to the browser.

## Scopes

The token issued to your MCP client carries scopes, and the server checks them on **every** tool call (not only at connection time):

| Scope | Allows |
|---|---|
| `amazon.read` | All read tools, including Ads configuration and `execute_sql` |
| `amazon.write` | Seller Central writes: listings, prices, stock, inbound plans |
| `amazon.ads` | Amazon Ads writes: campaigns, ad groups, keywords, targets, bids, budgets, state |

`amazon.write` and `amazon.ads` imply read access. A tool outside the granted scopes is not listed and cannot be called.

## Writes: the AI prepares, you approve

1. **Tool annotations.** Every tool declares `readOnlyHint`, `destructiveHint` and `openWorldHint` (MCP spec). Write tools are marked `readOnlyHint: false`, so Claude and ChatGPT ask for your permission before running them.
2. **Preview before every write.** The server instructs the assistant to show what will change (SKU/ASIN, value before → after, number of rows) and to wait for your explicit confirmation. Bulk changes are summarized before execution.
3. **Two-phase confirmation for headless clients.** For custom agents without a human "Allow" step in the client, the server can require a two-step commit: the first call returns a preview and a `confirm_id`; only a second call with the **same arguments** and that `confirm_id` executes. Different arguments invalidate the confirmation.
4. **Hard limits on the server** that no confirmation overrides: maximum rows per bulk call, maximum daily Ads budget per campaign, maximum price change versus the last known price.
5. **Read-after-write**: Ads writes return an explicit outcome and are verified by reading the entity back.

Vendooly is not an unattended autopilot: nothing is changed on Amazon "in the background" by the assistant.

## Tenant isolation

Every request is checked against the authorized organization, user, seller account and marketplace. One tenant cannot select another tenant's context. Agencies with several seller accounts work across them without mixing contexts: each call is bound to one outpost (one seller on one marketplace).

## Data minimization

- Tools return the operating context needed for the task, not a full export of the seller account.
- Order tools return order IDs, status, dates and totals, not buyer personal data. Dashboards and recaps return aggregates only.
- Customer return comments are treated as untrusted data, never as instructions.

## Your side

- Grant only the Amazon capabilities you need (Seller, Ads) and review access when people change roles.
- Check seller account, marketplace and affected rows before approving a write.
- Report suspected authorization issues promptly.

## Contacts and documents

- Security and responsible disclosure: security@vendooly.com
- Security page: https://vendooly.com/en/security/
- Trust center and tool manifest: https://vendooly.com/trust/ · https://vendooly.com/trust/capabilities.json
- Privacy policy: https://vendooly.com/legal/privacy.html
