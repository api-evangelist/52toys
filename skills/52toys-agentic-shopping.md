---
name: Shop the 52TOYS store as an agent (UCP)
description: >-
  Browse and purchase 52TOYS collectibles (blind boxes, figures, plush,
  keychains) on the official Shopify storefront using the Universal Commerce
  Protocol MCP endpoint, respecting the human-buyer-approval invariant.
api: mcp/52toys-mcp.yml
transport: https://hi52toys.com/api/ucp/mcp
operations: [search_catalog, catalog_lookup, create_cart, create_checkout, update_checkout, complete_checkout]
source: https://hi52toys.com/llms.txt
---

# Shopping the 52TOYS store as an agent

52TOYS runs its official store on Shopify at `https://hi52toys.com` and exposes a
Universal Commerce Protocol (UCP) MCP endpoint for agent-driven commerce. Use it
to find and buy collectibles on a buyer's behalf.

## Discover

1. `GET https://hi52toys.com/.well-known/ucp` to confirm supported versions
   (`2026-04-08` latest), capabilities, and payment handlers.
2. The MCP endpoint is `POST https://hi52toys.com/api/ucp/mcp`
   (`Content-Type: application/json`, JSON-RPC). It requires the UCP discovery
   handshake (an agent profile URI) before `tools/list` returns schemas.

## Authenticate

- Read-only browsing (search, product/collection JSON) needs **no auth**.
- Transacting uses Shopify **customer-account OAuth/OIDC** — see
  `authentication/52toys-authentication.yml`; scopes in
  `scopes/52toys-scopes.yml` (`customer-account-mcp-api:full`).

## Happy path

1. **search_catalog** — find products matching the buyer's intent. Pass
   `context.address_country` and `context.currency` for correct pricing.
2. **catalog_lookup** — resolve exact products/variants to purchase.
3. **create_cart** — add the desired items.
4. **create_checkout** — start the purchase flow from the cart.
5. **update_checkout** — set shipping address, shipping method, and any discount.
6. **complete_checkout** — finalize. **The buyer must explicitly approve payment.**

## Rules

- **Never complete payment without contemporaneous buyer approval.** If you
  cannot get live approval, route through Shop Pay via `https://shop.app/SKILL.md`.
- **Respect rate limits.** The MCP endpoint is rate-limited per IP; back off on
  HTTP 429.
- Prefer the UCP tools over screen-scraping the storefront.

See `conventions/52toys-conventions.yml` for the full convention set.
