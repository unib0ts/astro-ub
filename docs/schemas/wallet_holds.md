# `wallet_holds`

Tracks open reservations for active sessions.

```ts
WalletHold {
  _id: ObjectId
  accountId: ObjectId
  ownerId: ObjectId
  sessionId: ObjectId
  amount: number              // original hold (paise)
  captured: number            // sum of captures
  released: number
  status: "open" | "closed"
  idempotencyKey: string      // unique — usually session start key
  createdAt: Date
  updatedAt: Date
  closedAt?: Date
}
```

## Invariants

- `captured + released <= amount` (soft; top-ups may extend hold — see wallet doc)
- `status=closed` ⇒ no further capture
- Remaining on close: `amount - captured - released` must be released in same txn

## Indexes

- unique: `idempotencyKey`
- unique sparse: `sessionId` while one hold per session
- `{ status: 1, sessionId: 1 }`
