# Monorepo layout & Fastify plugin map

Target structure for Node.js + Fastify. pnpm workspaces + Turborepo (or nx). TypeScript everywhere.

## 1. Top-level tree

```
astro-platform/
├── apps/
│   ├── api-gateway/           # public edge
│   ├── core-api/              # auth, users, astrologers, sessions, reviews, admin
│   ├── wallet/                # money service
│   ├── chat/                  # wrap existing chat package
│   ├── call/                  # wrap existing video/call package
│   ├── notify/                # FCM consumer + thin HTTP
│   └── worker/                # billing ticks, saga compensations, outbox
├── packages/
│   ├── shared-config/         # env schema (zod), constants, price-tier codes
│   ├── shared-types/          # DTO types shared by OpenAPI generation
│   ├── shared-errors/         # AppError, error codes
│   ├── shared-auth/           # JWT verify/sign helpers, actor header types
│   ├── shared-logger/         # pino setup
│   ├── shared-mongo/          # Mongo client plugin factory
│   ├── shared-redis/          # Redis plugin factory
│   ├── shared-http/           # internal fetch client + idempotency helpers
│   ├── shared-events/         # event envelope + publisher interfaces
│   └── eslint-config/         # optional
├── docs/
│   ├── architecture/
│   └── schemas/
├── openapi/                   # optional generated dumps
├── scripts/
│   └── dev.ts
├── pnpm-workspace.yaml
├── turbo.json
├── package.json
├── tsconfig.base.json
└── .env.example
```

### Integrating existing chat / video packages

Prefer **not** rewriting them. Two options:

1. **Git submodule / private npm** depended on by `apps/chat` and `apps/call`  
2. **Copy into** `packages/chat-engine`, `packages/call-engine` and adapt adapters  

`apps/chat` stays a thin Fastify host: auth bridge (validate session media token via core-api or shared JWT secret), map internal room APIs, health checks.

## 2. pnpm workspace

```yaml
# pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"
```

```json
// package.json (root)
{
  "name": "astro-platform",
  "private": true,
  "packageManager": "pnpm@9",
  "scripts": {
    "dev": "turbo run dev --parallel",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "test": "turbo run test",
    "typecheck": "turbo run typecheck"
  }
}
```

## 3. Per-app Fastify bootstrap pattern

Every app follows the same shape:

```
apps/core-api/src/
  app.ts                 # buildApp(): FastifyInstance
  server.ts              # listen
  plugins/
    mongo.ts
    redis.ts
    auth.ts              # optional; gateway usually authenticates
    sensible.ts
  modules/
    users/
      index.ts           # fp plugin exporting routes
      routes.ts
      service.ts
      repo.ts
      schemas.ts         # zod / typebox
    astrologers/
    sessions/
    auth/
    reviews/
    admin/
  clients/
    wallet.ts            # HTTP to wallet
    chat.ts
    call.ts
    notify.ts
```

### `buildApp` sketch

```ts
import Fastify from "fastify";
import sensible from "@fastify/sensible";
import cors from "@fastify/cors";
import { mongoPlugin } from "@astro/shared-mongo";
import { redisPlugin } from "@astro/shared-redis";
import { usersModule } from "./modules/users/index.js";
import { sessionsModule } from "./modules/sessions/index.js";
// ...

export async function buildApp() {
  const app = Fastify({
    logger: true,
    requestIdHeader: "x-request-id",
    genReqId: (req) => (req.headers["x-request-id"] as string) || crypto.randomUUID(),
  });

  await app.register(sensible);
  await app.register(cors, { origin: false }); // public CORS only on gateway
  await app.register(mongoPlugin, { uri: env.MONGO_URI, dbName: "core" });
  await app.register(redisPlugin, { url: env.REDIS_URL });

  // Trust gateway identity headers on private network only
  app.addHook("onRequest", async (req) => {
    req.actor = {
      id: req.headers["x-actor-id"] as string | undefined,
      role: req.headers["x-actor-role"] as string | undefined,
    };
  });

  await app.register(usersModule, { prefix: "/v1/users" });
  await app.register(sessionsModule, { prefix: "/v1/sessions" });
  // ...

  app.get("/health", async () => ({ ok: true }));
  return app;
}
```

Use `fastify-plugin` (`fp`) so decorators are visible across encapsulations where intended.

## 4. API Gateway plugin map

```
apps/api-gateway/src/
  app.ts
  plugins/
    rate-limit.ts          # @fastify/rate-limit — stricter on /auth/otp
    jwt.ts                 # verify access token (JWKS or shared secret)
    proxy.ts               # @fastify/http-proxy or custom undici
    request-guard.ts       # strip spoofed x-actor-*; attach from JWT
  routes/
    auth.proxy.ts
    users.proxy.ts
    astrologers.proxy.ts
    sessions.proxy.ts
    wallet.proxy.ts
    chat.proxy.ts          # includes WS upgrade
    calls.proxy.ts
```

### Gateway request pipeline

```
onRequest:
  1. ensure request-id
  2. strip incoming x-actor-id / x-actor-role / x-actor-device-id
  3. if route.meta.auth !== public:
       verify JWT
       reject if role not allowed for route
       set x-actor-* toward upstream
  4. rate limit bucket by IP + actorId
preHandler (optional):
  blocklist check (redis) for banned tokens / blocked users
then:
  proxy to upstream
```

### Upstream map

| Prefix | Upstream | Auth |
|--------|----------|------|
| `/v1/auth/otp/*` | core-api | public |
| `/v1/auth/refresh` | core-api | public (refresh body) |
| `/v1/auth/logout` | core-api | access |
| `/v1/users` | core-api | user |
| `/v1/astrologers` | core-api | mixed (list public-ish with optional auth; me = astrologer) |
| `/v1/sessions` | core-api | user/astrologer |
| `/v1/reviews` | core-api | user |
| `/v1/wallet` | wallet | user/astrologer |
| `/v1/admin` | core-api | admin |
| `/v1/chat` | chat | session or actor JWT |
| `/v1/calls` | call | session or actor JWT |
| `/internal/*` | not exposed | — |

**Never** expose `/internal/*` on the gateway.

## 5. core-api module plugin map

| Module | Prefix | Key responsibilities |
|--------|--------|----------------------|
| `auth` | `/v1/auth` | OTP request/verify, refresh, logout |
| `users` | `/v1/users` | me, patch, family CRUD |
| `astrologers` | `/v1/astrologers` | list/search, me, presence, admin status |
| `catalog` | `/v1/catalog` | public price tiers (optional) |
| `sessions` | `/v1/sessions` | start/end/get/list; orchestrates wallet+chat+call |
| `reviews` | `/v1/reviews` | create; list by astrologer |
| `admin` | `/v1/admin` | approve/ban, tier assign |
| `internal` | `/internal` | validate media token, get session |

### sessions module dependencies

```
sessions/service
  → repo (sessions collection)
  → astrologers/repo (busy, gates)
  → clients.wallet.hold|capture|close|credit
  → clients.chat.createRoom|closeRoom
  → clients.call.create|end
  → clients.notify.enqueue
  → redis presence
```

Keep HTTP handlers thin; all orchestration in `service.ts`.

## 6. wallet plugin map

```
modules/
  accounts/     GET balance (by actor headers)
  holds/        internal hold/capture/close
  ledger/       GET history
  payments/     recharge + webhook
  internal/     service-token protected
```

Public via gateway: `/v1/wallet`, `/v1/wallet/ledger`, `/v1/wallet/recharge`, `/v1/wallet/webhooks/:provider`.

Internal: `/internal/holds`, `/internal/holds/:id/capture`, `/internal/holds/:id/close`, `/internal/credits`.

## 7. worker

Not a public HTTP app (optional health port).

**Queues (BullMQ):**

| Queue | Job | Producer |
|-------|-----|----------|
| `session-tick` | repeatable per active session or global scanner | on session start / cron |
| `notify` | FCM send | all services |
| `wallet-cache-sync` | update users.walletBalance | wallet events |
| `session-compensate` | closeHold if session create fails | sessions service |

**Global scanner alternative:** every 30s `sessions.find({ status: "active" })` and enqueue tick — simpler MVP than per-session repeatable jobs.

## 8. Shared auth package

```ts
// packages/shared-auth
export type ActorRole = "user" | "astrologer" | "admin" | "service";

export type AccessClaims = {
  sub: string;
  role: ActorRole;
  sid: string;     // refresh session id
  did?: string;    // deviceId
};

export type MediaClaims = {
  sid: string;     // consultation session id
  typ: "chat" | "audio" | "video";
  sub: string;     // actor
  role: "user" | "astrologer";
  peerId: string;
};
```

Access tokens minted by core-api auth module. Media tokens minted by sessions module. Chat/call verify media JWT with shared secret/public key — **no callback required on every WS message** (verify signature locally); optional introspect on room join only.

## 9. Env surface (per service)

```bash
# common
NODE_ENV=
PORT=
LOG_LEVEL=
REDIS_URL=
MONGO_URI=
JWT_ACCESS_SECRET=          # or asymmetric keys
JWT_MEDIA_SECRET=
SERVICE_TOKEN=              # s2s

# gateway
CORE_API_URL=
WALLET_URL=
CHAT_URL=
CALL_URL=

# core-api
WALLET_URL=
CHAT_URL=
CALL_URL=
NOTIFY_URL=
OTP_PROVIDER_*=
COMMISSION_PCT=25
MIN_SESSION_MINUTES=5

# wallet
PSP_RAZORPAY_KEY=
PSP_RAZORPAY_SECRET=
PSP_WEBHOOK_SECRET=

# notify
FCM_PROJECT_ID=
FCM_CLIENT_EMAIL=
FCM_PRIVATE_KEY=
```

Validate with zod at boot; fail fast.

## 10. Local compose

```yaml
# deploy/docker-compose.yml (sketch)
services:
  mongo:
  redis:
  gateway:
  core-api:
  wallet:
  chat:
  call:
  notify:
  worker:
```

Developers run `pnpm dev` with remote Atlas optional; compose for deps.

## 11. OpenAPI workflow

- Source of truth: `docs/architecture/01-openapi-mvp.yaml`
- Optionally generate types into `packages/shared-types` via `openapi-typescript`
- Fastify route schemas: TypeBox mirroring OpenAPI (manual for MVP is fine)
- Contract tests: schemathesis / dredd against gateway later

## 12. Coding conventions

- One module = one Fastify plugin file tree  
- No cross-module deep imports (`sessions` must not import `users/repo`; use `users/service` public functions or events)  
- Money math only in `wallet`  
- All external side effects after successful local commit where possible; else outbox  
- `Idempotency-Key` required on: OTP verify (optional), session start, recharge, session end  

## 13. First implementation order

1. Scaffold monorepo + shared packages + compose  
2. gateway JWT + proxy to stub core-api health  
3. core-api auth OTP + users + astrologers CRUD  
4. wallet accounts/ledger/hold/capture/close + tests  
5. sessions orchestration + worker tick  
6. wrap chat package with media JWT  
7. notify FCM  
8. call package  
9. admin + reviews  

Do not start with Kubernetes complexity; one VM/ECS service set is enough for MVP load testing.
