---
name: polymarket-clob-auth-trade
description: >
  Advanced procedural workflow for L1/L2 authentication, EIP-712 signing,
  and placing trades on Polymarket CLOB API.
---

# Polymarket Trading & Authentication Pipeline

You are an AI agent implementing a trading bot (Rust) for Polymarket. Trading requires a two-level authentication flow. Follow this pipeline strictly.

## Step 1: L1 Auth (Obtain API Keys)

- Create an EIP-712 signature using the wallet's private key.
- **EIP-712 Domain:** `name="Polymarket CTF Exchange", version="1", chainId=137` (must be Polygon).
- Send headers `POLY_ADDRESS`, `POLY_SIGNATURE`, `POLY_TIMESTAMP`, `POLY_NONCE` to generate/retrieve `POLY_API_KEY`, `secret`, and `passphrase`.

## Step 2: L2 Auth (HMAC-SHA256 for Trading)

Every POST/DELETE request to the CLOB (e.g. `/order`) requires an L2 signature.

- **Algorithm:** HMAC-SHA256 using your API `secret`.
- **String to sign:** `timestamp + method + path + body`.
- Pass the result in the `POLY_SIGNATURE` header along with `POLY_API_KEY` and `POLY_PASSPHRASE`.

## Step 3: Order Construction & EIP-712 Signing

Build the Order struct.

**CRITICAL TYPING RULES:**
1. `token_id` obtained from Gamma API as a string (`"217426..."`) must be cast to `U256` for the EIP-712 signature.
2. `signature_type`: set `0` for EOA (standalone wallet), `1` for POLY_PROXY (Magic Link), or `2` for GNOSIS_SAFE.
3. `expiration`: `u64` unix timestamp, or `0` for Good Till Cancelled.

## Step 4: Place Order

- Send the signed order JSON via `POST https://clob.polymarket.com/order`.
- Attach the L2 HMAC headers from Step 2.
