# Schemas

Canonical Mongo shapes for core domains. Aligns with the frozen user plan and extended session/wallet design.

## Collections by database

### `core`

- [users](./users.md)
- [astrologers](./astrologers.md)
- [sessions](./sessions.md)
- [reviews](./reviews.md)
- [price_tiers](./price_tiers.md)
- [auth_sessions](./auth_sessions.md)

### `wallet`

- [wallet_accounts](./wallet_accounts.md)
- [wallet_ledger](./wallet_ledger.md)
- [wallet_holds](./wallet_holds.md)
- [payment_orders](./payment_orders.md)

Chat/call schemas stay inside those packages; core only stores foreign ids (`chatRoomId`, `callId`).
