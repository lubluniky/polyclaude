---
name: polymarket-ws-streaming
description: >
  Procedural guide for managing Polymarket WebSockets (CLOB and RTDS)
  for real-time prices, orderbooks, and user fills.
---

# Polymarket WebSockets Architecture

You are an AI agent setting up real-time data streaming for Polymarket. The system has two distinct WebSocket servers. Choose the right one for the task.

## Step 1: Choose the WSS Endpoint

- **Option A (CLOB WS):** `wss://ws-subscriptions-clob.polymarket.com/ws/market` — Use for subscribing to specific orderbooks (`book`), granular price changes, and the private user channel (`user` fills). Max 500 assets.
- **Option B (RTDS WS):** `wss://ws-live-data.polymarket.com` — Use for broad feeds (`crypto_prices`, `activity`, aggregated `clob_market`).

## Step 2: Connection Management (Critical Architecture)

**CLOB WS LIMITATION:** CLOB WS does **not** support unsubscribe.

**Heuristic:** If the trading bot needs to switch the tracked market on CLOB WS, you MUST tear down the current TCP connection and open a new one with the new `assets_ids` list. Do not attempt to send an unsubscribe command.

## Step 3: Heartbeat (Ping/Pong)

- **CLOB WS:** Send the string message `"PING"` exactly every 10 seconds.
- **RTDS WS:** Send a standard WebSocket PING frame every 5 seconds.

## Step 4: Parsing `price_change`

When listening to CLOB WS, expect incremental updates.

**Note:** Format changed since September 2025. Messages now contain a `price_changes[]` array where each asset has current `best_bid` and `best_ask`. Update the local orderbook cache based on these deltas.
