# Astro Platform

Backend architecture and MVP contracts for an Astrotalk-style astrology consultation product.

| Client | Stack |
|--------|--------|
| User / Astrologer apps | Flutter (Android team) |
| Backend | Node.js + Fastify |
| Chat / Call | Existing packages as separate microservices |

## Docs

Published site (GitHub Pages source = `/docs` folder):

- Wallet flows: https://unib0ts.github.io/astro-ub/architecture/02-wallet-state-machine.html

| Doc | What it covers |
|-----|----------------|
| [Architecture overview](docs/architecture/00-overview.md) | Services, auth, data ownership, MVP scope |
| [OpenAPI MVP](docs/architecture/01-openapi-mvp.yaml) | Exact public + internal HTTP contracts |
| [Wallet & sessions](docs/architecture/02-wallet-state-machine.md) | Money state machine, holds, Mongo transactions |
| [Monorepo & Fastify](docs/architecture/03-monorepo-and-fastify.md) | Folder layout, plugins, wiring |
| [Open decisions](docs/architecture/04-open-decisions.md) | Stakeholder locks before coding |
| [Schemas](docs/schemas/README.md) | Mongo collections (users, astrologers, …) |

## Deployables (MVP)

```
api-gateway → core-api | wallet | chat | call | notify | worker
```

## Status

Architecture freeze in progress. Implementation not started in this repo yet.
