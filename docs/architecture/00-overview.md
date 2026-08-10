# Architecture overview

Principal-engineer reference for the Astro Platform backend. Complements the deep slices in this folder.

## Goals

- Per-minute consults between users and astrologers
- Wallet-first billing with an append-only ledger
- Reuse existing chat and video packages as isolated microservices
- Flutter clients (user + astrologer apps or one role-gated app)
- Ship an MVP without over-splitting services

## Consult modalities (interim — marketing may refine later)

Treat media as **two independent services**. A session is **one modality only** (no parallel dual billing for now).

| Session `type` | Media service | Rate field on `price_tiers` | Billing |
|----------------|---------------|-----------------------------|---------|
| `chat` | `chat` microservice | `chatRatePerMinute` | Per minute, one hold, one clock |
| `audio` | `call` microservice (voice) | `callRatePerMinute` | Per minute, one hold, one clock |
| `video` | `call` microservice (video) | `videoRatePerMinute` | Per minute, one hold, one clock |

Rules for now:

- User starts **either** a chat session **or** a call session (audio/video) — not both as one stacked bill.
- Voice and video share the **call** deployable; they differ only by `type` + rate snapshot.
- Chat package never opens a call; call package never stores chat messages.
- If marketing later wants chat+call in parallel with stacked rates, that becomes a session-policy change only — wallet hold/capture primitives stay the same.

## Deployable topology

```
Flutter apps
    │  HTTPS / WSS
    ▼
┌──────────────────┐
│   api-gateway    │  JWT, rate limit, route, request-id
└────────┬─────────┘
         │
    ┌────┴────┬──────────┬─────────┬──────────┐
    ▼         ▼          ▼         ▼          ▼
 core-api   wallet     chat      call      notify
    │         │          │         │
    └────┬────┘          │         │
         ▼               │         │
      worker ────────────┴─────────┘
   (billing ticks, webhooks reconcile, FCM jobs)

MongoDB (core | wallet | chat DBs)
Redis   (OTP, presence, locks, queues, refresh denylist)
```

| Service | Responsibility | Public? |
|---------|----------------|---------|
| `api-gateway` | Edge: TLS (or ALB), authn, RBAC route guards, proxy | Yes |
| `core-api` | Auth OTP issue/verify, users, astrologers, sessions, reviews, price tiers, admin | Via gateway |
| `wallet` | Accounts, ledger, holds, recharge, payouts later | Via gateway |
| `chat` | Rooms, messages, WS (existing package) | Via gateway |
| `call` | Signaling / media tokens (existing package) | Via gateway |
| `notify` | FCM (+ SMS templates later) | Internal + queue |
| `worker` | Session billing ticker, async consumers | Internal |

## Auth (summary)

- OTP on `mobile` + `role` (`user` | `astrologer` | `admin`)
- Access JWT 15–30m; refresh 30–90d with rotation
- Gateway validates JWT and injects `x-actor-id`, `x-actor-role`, `x-request-id`
- Chat/call join requires short-lived **session media token** minted by `core-api` after wallet hold succeeds
- Service-to-service: private network + `Authorization: Bearer <service-jwt>`

Details live in OpenAPI security schemes and wallet doc session flow.

## Data ownership

| Data | Owner | Notes |
|------|-------|-------|
| users, astrologers, sessions, reviews, price_tiers, auth_sessions | core-api DB | |
| wallet_accounts, ledger, payment_orders, holds | wallet DB | Money source of truth |
| rooms, messages | chat DB | |
| call history (optional) | call DB | Signaling mostly Redis |
| presence online/busy | Redis primary | Mongo lag for listing filters OK |
| OTP, refresh revoke, billing locks | Redis | |

`users.walletBalance` / `astrologers.walletBalance` are **display caches** only. Ledger is authoritative.

## Session money flow (one glance)

1. User starts session → core-api checks astrologer availability  
2. Resolve `ratePerMinute` from `priceTier` (snapshot on session)  
3. `wallet.hold` for `MIN_MINUTES * rate`  
4. Create session `active`, set astrologer busy  
5. If `type=chat` → `chat.createRoom`; if `audio|video` → `call.create`  
6. Worker captures per minute with idempotency key `sessionId:capture:N`  
7. End → release hold remainder, credit astrologer net of commission  

See [02-wallet-state-machine.md](./02-wallet-state-machine.md).

## MVP vs later

**MVP:** OTP auth, profiles, listing, recharge, chat consult + billing, basic audio if package ready, FCM, history, reviews, admin approve/ban.

**Later:** video polish, waitlist, payouts, coupons, kundli worker, split auth/session services.

## Locked product defaults (proposed)

| Decision | Value |
|----------|--------|
| Money unit | Integer paise (INR * 100) |
| `priceTier` | `bronze` \| `silver` \| `gold` |
| Modalities | Individual sessions: `chat` \| `audio` \| `video` (call svc = voice+video) |
| Parallel chat+call billing | Deferred — marketing TBD |
| Min start balance | 5 minutes at tier rate |
| Access token TTL | 15 minutes |
| Refresh TTL | 30 days, rotate on use |
| Commission | Config `%` snapshotted on session end |
| API version | `/v1` |

Confirm remaining rows with stakeholders before implementation freeze.
