# Vendooly MCP tools

This is the list of tools the Vendooly remote MCP server exposes to the assistant (Claude, ChatGPT or any MCP client). Names and access levels come from the production server code; the one-line descriptions are summaries.

**106 tools: 65 read, 41 write, plus a `ping` health check.** 44 tools are for Amazon Ads (Sponsored Products, Sponsored Brands, Sponsored Display) and 7 for Vendor Central.

Access legend:

- **Read**: reads data, changes nothing on Amazon. Annotated `readOnlyHint: true`.
- **Write**: changes something on Amazon (prices, stock, listings, bids, budgets, campaign state, inbound plans). Annotated `readOnlyHint: false`, so the client asks for your permission, and it runs only after your explicit approval. See [security.md](security.md).
- **Draft (non-binding)**: `generate_inbound_options` asks Amazon to generate packing/placement options for an inbound plan. Nothing is committed until `confirm_inbound_option`. It is counted with the reads in the totals above, but is flagged as non-read-only for clients.
- **Utility**: documentation, feedback, health check.

Notes:

- Every tool that works on a seller account accepts an optional `account_id` (the outpost, i.e. one seller on one marketplace; see `list_clients`).
- Sponsored Brands and Sponsored Display tools manage **existing** campaigns (read, bids, budgets, state). New campaigns can be created for Sponsored Products.
- Vendor Central tools appear only when your organization has a Vendor Central account connected.
- Some reports are asynchronous on Amazon's side: the first call may take 30-60 seconds.
- Machine-readable manifest of the same tools: https://vendooly.com/trust/capabilities.json

## Accounts and overview

| Tool | Access | What it does |
|---|---|---|
| `list_clients` | Read | List the connected Amazon outposts (one seller on one marketplace) and their IDs |
| `get_client_status` | Read | Status and details of the selected seller account |
| `get_dashboard_data` | Read | Aggregated KPI summary for one outpost (listing counts, 7-day revenue, issue count) |
| `get_all_dashboards` | Read | Aggregated KPI summary across all outposts of the organization |
| `get_account_health` | Read | Aggregated account health metrics: marketplace participation, order and cancellation rates |

## Analysis and history

| Tool | Access | What it does |
|---|---|---|
| `execute_sql` | Read | Read-only SQL over the Vendooly data-hub: daily sales, Ads performance, placements, settlement transactions, history |
| `weekly_recap` | Read | Operational recap: revenue, orders, FBA/FBM split, account health, top products by velocity |
| `diagnose_sales_drop` | Read | Compares the last N days with the previous period and returns likely causes plus suggested actions |
| `findings_list` | Read | Daily operational findings computed by a deterministic engine (Buy Box lost, stockout risk, Ads on low stock, negative margin) |

## Catalog and listings

| Tool | Access | What it does |
|---|---|---|
| `list_my_products` | Read | Own products: SKU, ASIN, title, price, FBA+MFN stock, fulfillment channel, sales velocity |
| `get_my_catalog_detail` | Read | Detail of one own product from the catalog cache |
| `search_catalog` | Read | Search the public Amazon catalog (best with ASIN/EAN/ISBN) |
| `get_catalog_item` | Read | Live catalog item lookup by ASIN |
| `get_product_types` | Read | Product type definition and required attributes |
| `get_listing` | Read | One listing: summaries, issues, offers, fulfillment availability, optional attributes |
| `get_listing_issues` | Read | Amazon issues for specific SKUs |
| `list_inactive_listings` | Read | Catalog-wide list of listings that are not for sale, with the reason (suppressed, removed, attribute rejected) |
| `create_listing` | **Write** | Create or replace a listing |
| `update_listing` | **Write** | Update an existing listing (title, bullets, attributes) |
| `delete_listing` | **Write** | Delete a listing |
| `bulk_update_listings` | **Write** | Bulk update of listing attributes (title, bullet points, patches) |
| `bulk_delete_listings` | **Write** | Bulk delete of offers, filtered by fulfillment channel |
| `update_item_highlights` | **Write** | Update the Item Highlights attribute on a listing (where the marketplace supports it) |
| `check_feed_status` | Read | Processing status and outcome summary of a submitted feed |

## Catalog audits and A+ Content

| Tool | Access | What it does |
|---|---|---|
| `audit_catalog_quality` | Read | Catalog quality audit |
| `audit_title_compliance` | Read | Checks every title against the Amazon title length limit |
| `audit_compliance` | Read | Compliance / GPSR audit of the catalog against the real attributes of each product type |
| `audit_aplus_coverage` | Read | Which ASINs have approved A+ Content and which do not |
| `list_aplus_documents` | Read | List A+ Content documents with their status |
| `get_aplus_document` | Read | Detail of one A+ Content document |

## Pricing and Buy Box

| Tool | Access | What it does |
|---|---|---|
| `get_my_pricing` | Read | Your own price per SKU |
| `get_competitive_pricing` | Read | Buy Box / Featured Offer price per ASIN |
| `get_buybox_prices` | Read | Bulk Buy Box export for a list of ASINs |
| `get_fee_estimate` | Read | Estimate Amazon fees (referral, FBA, closing) and net proceeds for a price |
| `update_price` | **Write** | Update the price of a SKU |
| `bulk_update_prices` | **Write** | Bulk price update |
| `reprice_to_buybox` | **Write** | Proposes Buy Box prices for given SKUs (never below your floor) and applies them after approval |

## Inventory, FBA and inbound

| Tool | Access | What it does |
|---|---|---|
| `get_fba_inventory` | Read | FBA inventory summaries |
| `update_mfn_quantity` | **Write** | Update the FBM (merchant-fulfilled) quantity of a SKU |
| `bulk_update_inventory` | **Write** | Bulk inventory update |
| `get_fba_customer_returns` | Read | Detailed FBA customer returns: SKU, reason, disposition, status, fulfillment center |
| `fba_inbound_discrepancies` | Read | Units received vs. unreconciled per shipment, cross-checked with reimbursements already granted |
| `get_fulfilled_shipments` | Read | Which fulfillment center (and country) each FBA order shipped from |
| `get_inbound_plans` | Read | List FBA inbound plans (Send to Amazon workflow) |
| `get_inbound_plan` | Read | Detail of one inbound plan, or status of a pending inbound operation |
| `create_inbound_plan` | **Write** | Create a draft FBA inbound plan (SKUs, quantities, ship-from address) |
| `generate_inbound_options` | Draft (non-binding) | Generate and list packing or placement options (with fees) for an inbound plan; non-binding drafts |
| `confirm_inbound_option` | **Write** | Confirm the chosen packing or placement option (placement creates the shipments) |
| `set_packing_information` | **Write** | Register box dimensions, weights and contents for an inbound plan |
| `cancel_inbound_plan` | **Write** | Cancel an inbound plan that has not shipped yet |

## Orders

| Tool | Access | What it does |
|---|---|---|
| `get_orders` | Read | Recent orders: IDs, status, dates, totals (no buyer data) |
| `get_order_items` | Read | Line items of one order |

## Amazon Ads: profiles, portfolios and checks

| Tool | Access | What it does |
|---|---|---|
| `ads_list_profiles` | Read | List Amazon Ads profiles |
| `ads_list_portfolios` | Read | List portfolios |
| `ads_create_portfolio` | **Write** | Create a portfolio |
| `ads_get_budget_usage` | Read | How much of today's daily budget each campaign has already spent |
| `ads_stockout_check` | Read | Active Sponsored Products ads on products close to stock-out |
| `ads_catalog_coverage` | Read | Every ASIN of the catalog with its live Sponsored Products coverage |

## Amazon Ads: Sponsored Products

| Tool | Access | What it does |
|---|---|---|
| `ads_list_sp_campaigns` | Read | Live campaign configuration: state, budget, targeting type, bidding |
| `ads_create_sp_campaign` | **Write** | Create a campaign (auto or manual) |
| `ads_update_sp_campaign` | **Write** | Update one or more campaigns (budget, state, bidding, placement modifiers) |
| `ads_delete_sp_campaign` | **Write** | Archive one or more campaigns |
| `ads_list_sp_ad_groups` | Read | List ad groups |
| `ads_create_sp_ad_group` | **Write** | Create an ad group (paused by default) |
| `ads_update_sp_ad_group` | **Write** | Update an ad group |
| `ads_list_sp_product_ads` | Read | List product ads with ASIN/SKU and campaign/ad group state |
| `ads_create_sp_product_ads` | **Write** | Create product ads |
| `ads_list_sp_keywords` | Read | Keywords with current bid and state |
| `ads_create_sp_keywords` | **Write** | Create keywords |
| `ads_update_sp_keyword_bid` | **Write** | Update keyword bids (single or batch) |
| `ads_update_sp_keyword_state` | **Write** | Enable or pause keywords (single or batch) |
| `ads_list_sp_negative_keywords` | Read | List negative keywords |
| `ads_create_sp_negative_keywords` | **Write** | Create negative keywords |
| `ads_delete_sp_negative_keywords` | **Write** | Delete negative keywords |
| `ads_list_sp_targets` | Read | Product/category targets and the auto-targeting groups |
| `ads_create_sp_product_targets` | **Write** | Create product or category targets |
| `ads_update_sp_target_bid` | **Write** | Update target bids (single or batch) |
| `ads_update_sp_target_state` | **Write** | Enable or pause targets (single or batch) |
| `ads_delete_sp_targets` | **Write** | Archive product targets |
| `ads_list_sp_negative_targets` | Read | List negative product targets |
| `ads_create_sp_negative_targets` | **Write** | Create negative product targets |
| `ads_delete_sp_negative_targets` | **Write** | Archive negative product targets |

## Amazon Ads: Sponsored Brands (existing campaigns)

| Tool | Access | What it does |
|---|---|---|
| `ads_list_sb_campaigns` | Read | Live state of existing Sponsored Brands campaigns |
| `ads_update_sb_campaign` | **Write** | Update budget, state and bidding of existing campaigns |
| `ads_list_sb_ad_groups` | Read | List ad groups |
| `ads_update_sb_ad_group` | **Write** | Update an existing ad group |
| `ads_list_sb_keywords` | Read | Keywords with current bid and state |
| `ads_update_sb_keyword_bid` | **Write** | Update keyword bids |
| `ads_update_sb_keyword_state` | **Write** | Enable, pause or archive keywords |
| `ads_list_sb_targets` | Read | Product targets with current bid and state |
| `ads_update_sb_target_bid` | **Write** | Update target bids |
| `ads_update_sb_target_state` | **Write** | Enable, pause or archive targets |

## Amazon Ads: Sponsored Display (existing campaigns)

| Tool | Access | What it does |
|---|---|---|
| `ads_list_sd_campaigns` | Read | Live state of existing Sponsored Display campaigns |
| `ads_update_sd_campaign` | **Write** | Update budget and state |
| `ads_list_sd_ad_groups` | Read | List ad groups with default bid |
| `ads_update_sd_ad_group` | **Write** | Update bid and state of ad groups |

## Vendor Central (shown only when a Vendor Central account is connected)

| Tool | Access | What it does |
|---|---|---|
| `generate_vendor_oauth_link` | Read | Authorization link to connect a Vendor Central account |
| `get_vendor_purchase_orders` | Read | Vendor purchase orders |
| `get_vendor_purchase_order_details` | Read | Detail of one purchase order |
| `get_vendor_po_summary` | Read | Aggregate purchase order summary |
| `get_vendor_invoices` | Read | Explains the SP-API limit: retail vendor invoices are not readable through the API |
| `get_vendor_remittances` | Read | Estimated amounts from purchase orders (the SP-API does not expose remittances for reading) |
| `get_vendor_trading_terms` | Read | Explains the SP-API limit: trading terms are not readable through the API |

## Help and utilities

| Tool | Access | What it does |
|---|---|---|
| `list_context_topics` | Utility | List the built-in guides (write confirmation flow, rate limits, multi-account, recipes) |
| `get_context_document` | Utility | Read one built-in guide |
| `send_feedback` | Utility | Send a bug report, idea or question to the Vendooly team |
| `ping` | Utility | Health check |
