# `auth_sessions`

Refresh-token sessions (store hash only).

```ts
AuthSession {
  _id: ObjectId
  actorId: ObjectId
  role: "user" | "astrologer" | "admin"
  deviceId: string
  refreshTokenHash: string
  expiresAt: Date
  revokedAt?: Date
  createdAt: Date
  updatedAt: Date
}
```

## Indexes

- `{ actorId: 1, role: 1, deviceId: 1 }`
- TTL optional on `expiresAt` (Mongo TTL index)
