# `reviews`

```ts
Review {
  _id: ObjectId
  sessionId: ObjectId         // unique
  userId: ObjectId
  astrologerId: ObjectId
  rating: number              // 1–5
  comment?: string
  createdAt: Date
}
```

## Indexes

- unique: `sessionId`
- `{ astrologerId: 1, createdAt: -1 }`
