# `users`

Frozen shape from stakeholder plan.

```ts
User {
  _id: ObjectId
  firstName: string
  lastName: string
  gender: string
  deviceId: string
  mobile: string              // unique login key
  dob: string
  tob: string
  birthPlace: string
  preferredLanguage: string[]
  appLanguage: string
  walletBalance: number       // display cache only; default 0
  isBlocked: boolean          // default false
  fcm: object
  profileImage: {
    url: string
    key?: string
    mime?: string
  }
  family: Array<{
    _id?: ObjectId
    relation: string
    firstName: string
    lastName?: string
    gender: string
    dob: string
    tob?: string
    birthPlace?: string
  }>
  createdAt: Date
  updatedAt: Date
}
```

## Indexes

- unique: `mobile`
- `isBlocked`
