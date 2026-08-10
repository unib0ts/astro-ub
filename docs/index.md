---
title: Astro Platform Docs
---

# Astro Platform

Backend architecture for the astrology consultation product (Fastify + Flutter).

## Architecture

| Doc | Description |
|-----|-------------|
| [Overview](./architecture/00-overview.html) | Services, auth, data ownership, MVP scope |
| [OpenAPI MVP](./architecture/01-openapi-mvp.html) | Public + internal HTTP contracts |
| [Wallet state machine](./architecture/02-wallet-state-machine.html) | Holds, captures, realtime billing flows |
| [Monorepo & Fastify](./architecture/03-monorepo-and-fastify.html) | Folder layout and plugin map |
| [Open decisions](./architecture/04-open-decisions.html) | Product locks before coding |

## Schemas

| Doc | Description |
|-----|-------------|
| [Schemas index](./schemas/) | Mongo collections by database |
| [Users](./schemas/users.html) | Frozen user shape |
| [Astrologers](./schemas/astrologers.html) | Astrologer profile + pricing tier |
| [Sessions](./schemas/sessions.html) | Consult orders |
| [Wallet accounts](./schemas/wallet_accounts.html) | Balances |
| [Wallet holds](./schemas/wallet_holds.html) | Session reserves |
| [Wallet ledger](./schemas/wallet_ledger.html) | Append-only money history |
| [Payment orders](./schemas/payment_orders.html) | Recharge / PSP |
| [Reviews](./schemas/reviews.html) | Ratings |
| [Price tiers](./schemas/price_tiers.html) | chat/call/video rates |
| [Auth sessions](./schemas/auth_sessions.html) | Refresh tokens |
