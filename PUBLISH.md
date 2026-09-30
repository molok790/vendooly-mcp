# PUBLISH — istruzioni per Filippo (file interno)

> Questo file serve solo a pubblicare il repo e fare le submission. **Cancellalo prima del
> push** (passo 1) oppure lascialo: non contiene dati riservati, ma è in italiano e rivolto
> a te. Consiglio: cancellarlo.

## 0. Controlli prima di pubblicare (5 minuti)

- [ ] README: numeri = FACTS.md del 23/9 (106 strumenti, 65 lettura / 41 azione, 44 Ads, 7 Vendor, prezzi da 39€/59€, trial 14 gg, gratis sotto 2.000€). Se FACTS cambia, aggiorna qui.
- [ ] `docs/tools.md`: 107 righe = 106 tool + `ping`, estratte dal codice (`TOOL_DEFS` meno `MODEL_HIDDEN`, + `execute_sql`) il 28/9. Uguale a `ionos/trust/capabilities.json`.
- [ ] `docs/connect-chatgpt.md`: i nomi dei menu di ChatGPT cambiano spesso (Connettori / App / Developer mode). Verifica su un account ChatGPT reale.
- [ ] `docs/connect-claude.md`: comando Claude Code `claude mcp add --transport http vendooly https://mcp.vendooly.com/mcp` — provalo una volta.
- [ ] `docs/security.md`: descrive la conferma in due fasi (`confirm_id`) come "can require" per i client headless, perché `safety.js` prevede `strict` (default) e `caps` (conferma lasciata all'host) e non ho potuto verificare quale modalità gira in produzione (in pm2/.env sul VPS non risulta la variabile → sarebbe `strict`). Se vuoi dirlo diversamente, è lì.
- [ ] Il badge "MCP Registry" punta alla ricerca del registry ufficiale: l'entry `com.vendooly/vendooly` è attiva dal 5/8/2026 (verificato 28/9).
- [ ] Niente cellulare, IBAN, gmail tecnico, nomi clienti, vecchia società: controllato. Unico contatto: hello@ (+ security@ nella pagina sicurezza).

## 1. Crea il repo e pubblica

```bash
cd /home/filippo/work/vendooly-mcp-docs
git rm -q PUBLISH.md && git commit -qm "chore: remove internal publish notes"   # consigliato
gh repo create molok790/vendooly-mcp --public --source . --push \
  --description "Remote MCP server for Amazon Seller Central (SP-API) and Amazon Ads in ChatGPT and Claude. Docs only; the service is hosted." \
  --homepage "https://vendooly.com/en/"
gh repo edit molok790/vendooly-mcp --add-topic mcp,mcp-server,remote-mcp,model-context-protocol,amazon,amazon-seller,seller-central,sp-api,amazon-ads,amazon-ppc,fba,chatgpt,claude,ecommerce
```

URL del repo: https://github.com/molok790/vendooly-mcp

## 2. MCP Market — coda gratuita

- Pagina: https://mcpmarket.com/submit
- Campi: **GitHub repository URL** = `https://github.com/molok790/vendooly-mcp` · **Email** = `hello@vendooly.com`
- Scegli la **coda gratuita** (4-6 settimane). Il fast-track è 29 $ una tantum (24 h + badge): solo se decidi di pagarlo.

## 3. mcp.directory — gratuito

- Pagina: https://mcp.directory/submit
- **GitHub Repository URL** = `https://github.com/molok790/vendooly-mcp` (obbligatorio; legge da solo nome, descrizione, licenza e README)
- Short description (facoltativa): `MCP connector that brings Amazon Seller Central and Amazon Ads into ChatGPT and Claude via official Amazon APIs. The AI prepares, you approve every change.`
- Email: `hello@vendooly.com`. npm / PyPI: lasciare vuoti.
- Revisione dichiarata entro 24 ore.

## 4. awesome-remote-mcp-servers — pull request

Regole lette dal loro `CONTRIBUTING.md` (28/9/2026): una riga nella tabella del README,
colonne `Name | Category | URL | Authentication | Maintainer`; autenticazione "OAuth 2.0
following the MCP spec". Nel README la notazione usata è `OAuth2.1` (con 🔐 solo per chi
**non** supporta la Dynamic Client Registration: noi la supportiamo, quindi senza 🔐).
Categoria: esiste già `E-Commerce` (Mercado Libre). Le righe nuove si aggiungono in fondo
alla tabella "Remote MCP Server List" (oggi l'ultima è Wolfram, riga ~189).

**Riga esatta:**

```markdown
| Vendooly | E-Commerce | `https://mcp.vendooly.com/mcp` | OAuth2.1 | [Vendooly](https://vendooly.com) |
```

**Comandi:**

```bash
cd /tmp && gh repo fork jaw9c/awesome-remote-mcp-servers --clone && cd awesome-remote-mcp-servers
git checkout -b add-vendooly
# inserisci la riga subito dopo l'ultima riga della tabella (quella di Wolfram):
sed -i '/^| Wolfram | Productivity |/a | Vendooly | E-Commerce | `https://mcp.vendooly.com/mcp` | OAuth2.1 | [Vendooly](https://vendooly.com) |' README.md
git diff    # controlla che sia UNA riga, dentro la tabella
git commit -am "Add Vendooly (Amazon Seller Central + Amazon Ads)"
git push -u origin add-vendooly
gh pr create --repo jaw9c/awesome-remote-mcp-servers --title "Add Vendooly remote MCP server (Amazon Seller Central + Amazon Ads)" --body-file - <<'EOF'
Adds **Vendooly**, a remote MCP server that connects ChatGPT, Claude and other MCP clients to an Amazon seller account through the official Amazon APIs (Selling Partner API for Seller Central / Vendor Central, Amazon Ads API).

- **URL:** `https://mcp.vendooly.com/mcp` (streamable HTTP)
- **Authentication:** OAuth 2.1 per the MCP spec, with Dynamic Client Registration and PKCE (S256) — just paste the URL in the client
- **Maintainer:** Vendooly S.r.l. (official, the company that operates the service) — https://vendooly.com
- **Docs repo:** https://github.com/molok790/vendooly-mcp
- **Official MCP Registry:** `com.vendooly/vendooly` (active since August 2026)

Why it fits the quality criteria:
- Official and production: hosted multi-tenant service used daily by Amazon sellers and agencies on EU, NA and FE marketplaces.
- Security: server-side OAuth scope enforcement (`amazon.read` / `amazon.write` / `amazon.ads`), MCP tool annotations on every tool, explicit human approval before any write, server-side hard limits on bulk size, Ads budgets and price changes. Security page: https://vendooly.com/en/security/ · tool manifest: https://vendooly.com/trust/capabilities.json
- Support: hello@vendooly.com, public wiki in 5 languages (https://vendooly.com/wiki/en/), in-product feedback tool.
EOF
```

> Nota: il body della PR non contiene numeri di utenti né nomi di clienti (FACTS: "decine di
> seller e agenzie" solo se richiesto). Se preferisci aggiungerlo: "over 40 registered users".

## 5. (Facoltativo) collegare il repo all'entry del registry ufficiale

Il `server.json` di questo repo è **identico** a quello pubblicato (`mcp-registry/server.json`
nel repo Vendooly, versione 1.0.0). Per far comparire il link al repo nel registry:

1. aggiungi in `server.json` (qui e in `Vendooly/mcp-registry/server.json`):
   ```json
   "repository": { "url": "https://github.com/molok790/vendooly-mcp", "source": "github" },
   ```
   e porta `"version"` a `"1.0.1"`;
2. ripubblica con `mcp-publisher login http --domain vendooly.com ...` + `mcp-publisher publish`
   (serve la chiave privata dell'autenticazione di dominio, quella dietro
   `ionos/.well-known/mcp-registry-auth`: la hai tu, non va nel repo).

Non è necessario per MCP Market / mcp.directory / awesome list.

## 6. Dopo

- Aggiorna Notion (Vendooly → Marketing 10K → directory) con: repo URL, data invio, stato per ciascuna delle 3 submission.
- Aggiorna `marketing/autopilot/outreach/submission-kit/piano-invii.md` nel repo Vendooly.
