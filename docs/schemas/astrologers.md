# `astrologers`

```ts
Astrologer {
  _id: ObjectId
  firstName: string
  lastName: string
  gender: string
  mobile: string              // unique login
  email?: string
  dob?: string
  profileImage: { url: string; key?: string; mime?: string }

  displayName: string
  bio: string
  experienceYears: number
  skills: string[]
  languages: string[]
  specializations: string[]

  priceTier: string           // bronze | silver | gold — rates via price_tiers

  isOnline: boolean           // mirrored from Redis periodically
  isBusy: boolean
  blockedBy: ObjectId[]       // userIds
  isBanned: boolean
  isVerified: boolean
  status: "pending" | "approved" | "rejected" | "suspended"
  approvedAt?: Date
  rejectedReason?: string

  ratingAvg: number
  ratingCount: number
  totalOrders: number

  deviceId: string
  fcm: object

  walletBalance: number       // display cache of earnings; default 0

  createdAt: Date
  updatedAt: Date
}
```

## Indexes

- unique: `mobile`
- `status`, `isOnline`, `priceTier`, `isBanned`
- compound listing (example): `{ status: 1, isBanned: 1, isOnline: -1, ratingAvg: -1 }`
