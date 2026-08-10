# `sessions`

Consultation order — billing authority lives with wallet; this doc snapshots rates and outcomes.

**Interim modality rule:** one session is exactly one of `chat` | `audio` | `video`.

| `type` | Microservice | Rate from `price_tiers` |
|--------|--------------|-------------------------|
| `chat` | chat | `chatRatePerMinute` |
| `audio` | call (voice) | `callRatePerMinute` |
| `video` | call (video) | `videoRatePerMinute` |

Parallel chat+call stacked billing is deferred (marketing TBD). Wallet path is identical for all types.

```ts
Session {
  _id: ObjectId
  type: "chat" | "audio" | "video"
  userId: ObjectId
  astrologerId: ObjectId
  priceTier: string
  ratePerMinute: number       // paise; snapshot at start from matching tier field
  commissionPct: number       // snapshot at end (or start)
  status: "pending" | "active" | "ended" | "cancelled" | "failed"
  startedAt?: Date
  endedAt?: Date
  endReason?: "user" | "astrologer" | "low_balance" | "system" | "error"
  durationSec: number
  billedMinutes: number
  totalCharged: number        // user debit (paise)
  astrologerEarning: number
  platformFee: number
  walletHoldId?: ObjectId
  chatRoomId?: string         // set when type=chat
  callId?: string             // set when type=audio|video
  mediaTokenJti?: string      // for revoke
  createdAt: Date
  updatedAt: Date
}
```

## Indexes

- `{ userId: 1, createdAt: -1 }`
- `{ astrologerId: 1, createdAt: -1 }`
- `{ status: 1, astrologerId: 1 }`
- `{ status: 1, startedAt: 1 }` // worker scan of actives
