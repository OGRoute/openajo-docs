# Events

Every state change emits an event. These are the integration surface for indexers and off-chain services — the OpenAjo [indexer](../developers/api.md) is built entirely by folding them. Topics are two `symbol` values; the table shows the data tuple.

## `circle` events

| Topics | Data | Emitted when |
| --- | --- | --- |
| `("circle", "create")` | `(id: u32, creator: Address)` | A circle is created |
| `("circle", "join")` | `(id: u32, member: Address)` | A member joins |
| `("circle", "start")` | `(id: u32)` | The circle fills and activates |
| `("circle", "contrib")` | `(id: u32, member: Address, cycle: u32)` | A member contributes |
| `("circle", "slash")` | `(id: u32, member: Address, amount: i128)` | A missed contribution is slashed from a deposit |
| `("circle", "default")` | `(id: u32, member: Address)` | A member's deposit is exhausted and they default |
| `("circle", "payout")` | `(id: u32, recipient: Address, amount: i128, cycle: u32)` | The pot is paid to a member |
| `("circle", "complete")` | `(id: u32)` | The circle completes |
| `("circle", "cancel")` | `(id: u32)` | The creator cancels an open circle |

## `reputation` events

| Topics | Data | Emitted when |
| --- | --- | --- |
| `("rep", "complete")` | `(member: Address)` | A completion is recorded |
| `("rep", "default")` | `(member: Address)` | A default is recorded |

## Consuming events

Fetch them from Soroban RPC with `getEvents`, filtered to the two contract IDs:

```ts
const page = await server.getEvents({
  startLedger,
  filters: [{
    type: "contract",
    contractIds: [CIRCLE_CONTRACT_ID, REPUTATION_CONTRACT_ID],
  }],
  limit: 100,
});
```

The SDK's `decodeEvent(topics, value)` turns each raw event into a typed `OpenAjoEvent`. See the [SDK reference](../developers/sdk.md).

> Event topic and data shapes are a compatibility contract. Changing them is a breaking change for every indexer, so treat it as a coordinated change across the contract and app repos.

## A note on completeness

Events report _that_ something happened, not the full resulting state. The OpenAjo indexer uses events to learn **which** circles changed, then re-reads current state from the contract via simulation. This keeps the off-chain database from ever drifting from on-chain truth. Build the same way if you index OpenAjo.
