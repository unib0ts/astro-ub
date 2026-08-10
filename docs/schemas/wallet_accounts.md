# `wallet_accounts`

```ts
WalletAccount {
  _id: ObjectId
  ownerId: ObjectId
  ownerType: "user" | "astrologer"
  balance: number             // available paise
  held: number                // reserved paise
  currency: "INR"
  version: number             // optimistic concurrency
  createdAt: Date
  updatedAt: Date
}
```

## Indexes

- unique compound: `{ ownerType: 1, ownerId: 1 }`
