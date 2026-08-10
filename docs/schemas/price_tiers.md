# `price_tiers`

```ts
PriceTier {
  _id: ObjectId
  code: string                // bronze | silver | gold
  chatRatePerMinute: number   // paise
  callRatePerMinute: number
  videoRatePerMinute: number
  currency: "INR"
  isActive: boolean
  updatedAt: Date
}
```

## Indexes

- unique: `code`
