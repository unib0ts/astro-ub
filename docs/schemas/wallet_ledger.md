# `wallet_ledger`

Append-only. Never update or delete rows in application code.

```ts
LedgerEntry {
  _id: ObjectId
  accountId: ObjectId
  ownerId: ObjectId
  ownerType: "user" | "astrologer"
  type: "credit" | "debit" | "hold" | "release" | "capture"
  amount: number              // always > 0
  balanceAfter: number        // available after this entry
  heldAfter: number
  reason: "recharge" | "session_hold" | "session_capture" | "session_release"
       | "session_earning" | "payout" | "refund" | "promo" | "adjustment"
  refType: "payment" | "session" | "payout" | "admin"
  refId: ObjectId | string
  idempotencyKey: string      // unique globally
  meta?: object
  createdAt: Date
}
```

## Indexes

- unique: `idempotencyKey`
- `{ accountId: 1, createdAt: -1 }`
- `{ refType: 1, refId: 1 }`
