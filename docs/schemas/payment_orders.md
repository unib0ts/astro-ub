# `payment_orders`

User wallet recharge via PSP (Razorpay / Cashfree / …).

```ts
PaymentOrder {
  _id: ObjectId
  userId: ObjectId
  amount: number              // paise
  currency: "INR"
  provider: "razorpay" | "cashfree"
  providerOrderId: string
  providerPaymentId?: string
  status: "created" | "paid" | "failed" | "expired"
  idempotencyKey: string      // unique
  rawWebhook?: object
  creditedLedgerId?: ObjectId
  createdAt: Date
  updatedAt: Date
}
```

## Indexes

- unique: `idempotencyKey`
- unique: `providerOrderId`
- `{ userId: 1, createdAt: -1 }`
- `{ status: 1 }`
