# Open decisions (lock before build)

| # | Topic | Proposal | Status |
|---|--------|----------|--------|
| 1 | `priceTier` values | `bronze` \| `silver` \| `gold` | Proposed |
| 2 | Money unit | Integer paise | Proposed — strong recommend |
| 3 | Min minutes to start | 5 | Proposed |
| 4 | Charge timing | Bill at start of each minute (incl. minute 0) | Proposed |
| 5 | Commission | Config `%`, snapshot on session end | Proposed |
| 6 | One vs two Flutter apps | Backend role-scoped either way | Product |
| 7 | Wallet UI balance | Show `available` + `held` separately; money screens use `GET /v1/wallet` | Proposed |
| 8 | PSP | Razorpay first | Proposed |
| 9 | Chat/call package integration | Private npm / existing repo adapter apps | Needs path from team |
| 10 | Project name | `astro-platform` under `~/Projects` | Provisional |
| 11 | Consult modalities | **Individual services for now:** `chat` alone; `audio`/`video` via **call** service. One session = one type = one rate clock. | Interim lock |
| 12 | Parallel chat + call (stacked rates) | Marketing to finalize later; do not build dual meters yet | Deferred |

When locked, update this table and bump overview “Locked product defaults”.
