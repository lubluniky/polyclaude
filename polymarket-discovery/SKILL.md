---
name: polymarket-market-discovery
description: >
  Procedural workflow for finding markets, extracting token IDs, and reading
  orderbooks on Polymarket using Gamma and CLOB APIs. Read-only, no auth required.
---

# Polymarket Market Discovery & Orderbook Retrieval

You are an AI agent. Your task is to find markets on Polymarket and retrieve orderbook data. The API is read-only (no auth required).

## Step 1: Market Search (Gamma API)

To find a market or event, use the Gamma API.

- **URL:** `https://gamma-api.polymarket.com/markets`
- **Params:** `active=true`, `closed=false`, `order=volume` (or `tag=...`).
- **Action:** Make a GET request and parse the JSON response.

**CRITICAL HEURISTIC (Sanity Check):**
In the Gamma API response, strictly distinguish two IDs:
1. `condition_id` (starts with `0x...`) — on-chain ID of the entire market.
2. `token_id` (numeric string, e.g. `"21742633..."`) — ID of a specific outcome (Yes or No) inside the `tokens` array.

**You always need `token_id` for the orderbook and trading!**

## Step 2: Orderbook Retrieval (CLOB API)

Use the CLOB API to get prices and liquidity.

- **URL:** `https://clob.polymarket.com/book?token_id={token_id}`
- **Action:** Make a GET request passing the `token_id` (NOT `condition_id`).
- Expected response contains `bids` and `asks` arrays with objects `{ "price": "0.52", "size": "150.5" }`.

## Step 3: Wallet Positions (Data API)

- **URL:** `https://data-api.polymarket.com/positions?user={address}`
- Returns current open positions for the given user address.
