# Vendooly — Amazon Seller Central and Amazon Ads inside ChatGPT and Claude

[![Website](https://img.shields.io/badge/website-vendooly.com-6C2CF5)](https://vendooly.com/en/)
[![MCP endpoint](https://img.shields.io/badge/MCP-remote%20%C2%B7%20streamable%20HTTP-071A3D)](#endpoint)
[![Auth](https://img.shields.io/badge/auth-OAuth%202.1-FF7A1A)](docs/security.md)
[![MCP Registry](https://img.shields.io/badge/MCP%20Registry-com.vendooly%2Fvendooly-24105A)](https://registry.modelcontextprotocol.io/v0/servers?search=vendooly)
[![Docs license](https://img.shields.io/badge/docs-CC%20BY%204.0-5B6275)](LICENSE)

Vendooly is a **remote MCP server** that connects the AI assistant you already use (ChatGPT or Claude) to your Amazon seller account: **Seller Central through the official Selling Partner API (SP-API) and Amazon Ads through the official Amazon Ads API**.

You ask in plain language: why did sales drop this week, which SKUs will stock out, which keywords spend without selling, where did I lose the Buy Box. The assistant reads your real data through Vendooly and, when you ask it to act, prepares the change and waits for your OK.

**Vendooly is not an AI of its own and not another dashboard.** It is the action layer between your AI and Amazon: defined, bounded tools, official APIs, human approval before every write.

> This repository contains **documentation only**. The Vendooly service is proprietary and hosted by Vendooly S.r.l.; there is nothing to install or self-host.

## At a glance

| | |
|---|---|
| Endpoint | `https://mcp.vendooly.com/mcp` |
| Transport | Remote, streamable HTTP |
| Auth | OAuth 2.1 with Dynamic Client Registration and PKCE (S256) |
| MCP Registry ID | `com.vendooly/vendooly` |
| Clients | Claude, ChatGPT, any MCP client that supports remote servers with OAuth |
| Amazon APIs | Selling Partner API (Seller Central, Vendor Central), Amazon Ads API |
| Tools | 106 (65 read, 41 write) + `ping`; 44 for Amazon Ads, 7 for Vendor Central |
| Languages | English, Italian, German, Spanish, French |
| Website | https://vendooly.com/en/ |
| Pricing | https://vendooly.com/pricing.html |
| Docs (wiki) | https://vendooly.com/wiki/en/ |
| Support | hello@vendooly.com |

## Endpoint

```
https://mcp.vendooly.com/mcp
```

Remote server, streamable HTTP transport, OAuth 2.1. Your client discovers the authorization server automatically, registers itself (Dynamic Client Registration) and opens the Vendooly consent page in the browser. You never paste an Amazon password or an API key into the chat.

Before connecting an assistant you need a Vendooly account with at least one Amazon seller account authorized:

1. Sign up at https://vendooly.com/signup.html (14-day free trial).
2. In the app, connect Seller Central with Amazon's official OAuth. One authorization covers all marketplaces of a region (EU, NA, FE).
3. Optionally connect Amazon Ads: it is a separate authorization.

## Connect it

**Claude** (plans that support custom connectors): Settings → Connectors → *Add custom connector* → paste `https://mcp.vendooly.com/mcp` → authorize. Full guide: [docs/connect-claude.md](docs/connect-claude.md).

**ChatGPT** (plans that support connectors / apps): Settings → Connectors (Apps) → *Create* → paste `https://mcp.vendooly.com/mcp`, authentication OAuth → authorize and enable it in the chat. Full guide: [docs/connect-chatgpt.md](docs/connect-chatgpt.md).

**Other MCP clients and custom agents**: point any client that supports remote MCP servers with OAuth to the same URL. The official MCP SDKs handle registration, consent and token refresh.

First message to check that it works:

> Now that you are connected to Vendooly, tell me which seller accounts and marketplaces you can see and what you can help me with.

## Security in short

- **Official Amazon OAuth**: no Seller Central passwords, no credentials in the chat. Amazon tokens stay encrypted on the server and never reach the model.
- **The AI prepares, you approve**: every write (prices, stock, listings, bids, budgets, campaign state) is shown first as a before → after preview and runs only after your explicit confirmation.
- **Read and write are separate**: every tool carries MCP annotations (`readOnlyHint`, `destructiveHint`), so Claude and ChatGPT ask your permission before a write tool runs.
- **Scopes enforced on the server**: `amazon.read`, `amazon.write`, `amazon.ads`.
- **Server-side hard limits** on bulk operations, Ads budgets and price changes that no confirmation can override.
- **Tenant isolation**: every call is bound to your organization, seller account and marketplace.

Details: [docs/security.md](docs/security.md).

## Tools

106 tools grouped by area (full table with read/write and description: [docs/tools.md](docs/tools.md)).

- **Accounts and overview** (5): `list_clients`, `get_client_status`, `get_dashboard_data`, `get_all_dashboards`, `get_account_health`
- **Analysis and history** (4): `execute_sql`, `weekly_recap`, `diagnose_sales_drop`, `findings_list`
- **Catalog and listings** (15): `list_my_products`, `get_my_catalog_detail`, `search_catalog`, `get_catalog_item`, `get_product_types`, `get_listing`, `get_listing_issues`, `list_inactive_listings`, `create_listing`, `update_listing`, `delete_listing`, `bulk_update_listings`, `bulk_delete_listings`, `update_item_highlights`, `check_feed_status`
- **Catalog audits and A+ Content** (6): `audit_catalog_quality`, `audit_title_compliance`, `audit_compliance`, `audit_aplus_coverage`, `list_aplus_documents`, `get_aplus_document`
- **Pricing and Buy Box** (7): `get_my_pricing`, `get_competitive_pricing`, `get_buybox_prices`, `get_fee_estimate`, `update_price`, `bulk_update_prices`, `reprice_to_buybox`
- **Inventory, FBA and inbound** (13): `get_fba_inventory`, `update_mfn_quantity`, `bulk_update_inventory`, `get_fba_customer_returns`, `fba_inbound_discrepancies`, `get_fulfilled_shipments`, `get_inbound_plans`, `get_inbound_plan`, `create_inbound_plan`, `generate_inbound_options`, `confirm_inbound_option`, `set_packing_information`, `cancel_inbound_plan`
- **Orders** (2): `get_orders`, `get_order_items`
- **Amazon Ads: profiles, portfolios and checks** (6): `ads_list_profiles`, `ads_list_portfolios`, `ads_create_portfolio`, `ads_get_budget_usage`, `ads_stockout_check`, `ads_catalog_coverage`
- **Amazon Ads: Sponsored Products** (24): campaigns, ad groups, product ads, keywords, negative keywords, product targets and negative targets — list, create, update bids/state, archive
- **Amazon Ads: Sponsored Brands, existing campaigns** (10): campaigns, ad groups, keywords and targets — list, bids, budgets, state
- **Amazon Ads: Sponsored Display, existing campaigns** (4): campaigns and ad groups — list, bids, budgets, state
- **Vendor Central** (7, shown only when a Vendor Central account is connected): purchase orders, PO details and summary, plus explicit answers where the SP-API does not expose invoices, remittances or trading terms
- **Help and utilities** (4): `list_context_topics`, `get_context_document`, `send_feedback`, `ping`

## 10 example prompts

1. *How much did I sell in the last 7 and 30 days, by marketplace?*
2. *Why did my sales drop this week compared with last week?*
3. *Which products are at risk of stocking out in FBA in the next weeks?*
4. *Where did I lose the Buy Box this week, and on which ASINs?*
5. *Which Sponsored Products keywords spent money in the last 14 days without a single sale?*
6. *Am I paying for ads on products that are about to run out of stock?*
7. *Which of my listings are inactive or suppressed, and why?*
8. *Check all my titles against Amazon's title length limit and list the ones to fix.*
9. *Pause the keywords you just found. Show me the list before doing it.*
10. *Reprice SKU X to win the Buy Box, never below my floor price. Show me before → after first.*

More prompts, grouped by area: [docs/prompts.md](docs/prompts.md). A library of 27 ready prompts in 5 languages is available inside the Vendooly app.

## Pricing

14-day free trial of the full Seller + Ads plan. Free under €2,000/month of Amazon revenue. Seller from €39/month, Seller + Ads from €59/month, + VAT; the price follows your 30-day Amazon revenue band. Current prices and details: **https://vendooly.com/pricing.html**.

## FAQ

**Is Vendooly an AI model or a chatbot?**
No. Vendooly does not bring its own AI. It gives the assistant you already use (ChatGPT, Claude) controlled access to your Amazon data and actions.

**Can the AI change my prices or campaigns on its own?**
No. Reading needs no confirmation; changing always does. Every write shows a before → after preview and runs only after your explicit OK. Bulk operations, Ads budgets and price changes also have server-side hard limits.

**Do I have to register as an Amazon developer?**
No. You authorize Vendooly with Amazon's official OAuth from the Vendooly app, like any other Amazon-integrated software.

**Which Amazon marketplaces are supported?**
The marketplaces of the EU, NA and FE regions that your seller account sells on and that the official APIs reach. One OAuth authorization covers one region.

**Does it work for agencies with several seller accounts?**
Yes. One Vendooly organization can manage several seller accounts and marketplaces; each request stays bound to the selected one.

**Does it read my customers' personal data?**
The order tools return order IDs, status, dates and totals, not buyer data. Aggregated tools (dashboards, recaps) return aggregates only.

**Is it an official Amazon, OpenAI or Anthropic product?**
No. Vendooly is an independent product by Vendooly S.r.l. that uses the official, public Amazon APIs and the open MCP standard.

**Where is the source code?**
The server is proprietary and hosted. This repository documents how to connect and what the tools do.

## Support

- Email: **hello@vendooly.com**
- Security reports: security@vendooly.com
- Documentation (wiki, 5 languages): https://vendooly.com/wiki/en/
- Security and trust: https://vendooly.com/en/security/ · https://vendooly.com/trust/
- Privacy: https://vendooly.com/legal/privacy.html

---

## In breve (IT)

Vendooly è un **server MCP remoto** che collega ChatGPT e Claude al tuo seller account Amazon: Seller Central con le **API ufficiali SP-API** e Amazon Ads con le **API ufficiali Amazon Ads**. Non è un'IA proprietaria né una dashboard: usa l'IA che già usi. **L'IA prepara, tu approvi**: ogni modifica (prezzi, stock, listing, bid, budget, stato campagne) mostra l'anteprima prima → dopo e parte solo con la tua conferma esplicita.

- Endpoint: `https://mcp.vendooly.com/mcp` (remoto, streamable HTTP, OAuth 2.1)
- Claude: Impostazioni → Connettori → *Aggiungi connettore personalizzato* → incolla l'URL → autorizza
- ChatGPT: Impostazioni → Connettori / App → *Crea* → incolla l'URL, autenticazione OAuth → autorizza
- 106 strumenti (65 di lettura, 41 di azione), 44 per Amazon Ads, 7 per Vendor Central
- Prova gratuita di 14 giorni; gratis sotto 2.000€/mese di fatturato Amazon. Prezzi: https://vendooly.com/pricing.html
- Guida in italiano: https://vendooly.com/guida.html · Wiki: https://vendooly.com/wiki/
- Contatto: hello@vendooly.com

---

Vendooly S.r.l. · Via Alessandro Manzoni 58, 04100 Latina (LT), Italy · VAT IT03382570590 · REA LT-337578

Amazon, Seller Central and Amazon Ads are trademarks of Amazon.com, Inc. or its affiliates. ChatGPT is a trademark of OpenAI; Claude is a trademark of Anthropic. Vendooly is not affiliated with, endorsed or certified by Amazon, OpenAI or Anthropic.

Documentation licensed under [CC BY 4.0](LICENSE). The Vendooly service is proprietary.
