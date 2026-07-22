# Reputation

A security [deposit](deposits-and-slashing.md) prices default within a single circle. Reputation is what makes repeat defaulting expensive across circles — it is the memory the traditional system never had.

## What is recorded

The `reputation` contract keeps two counters per address:

| Counter | Incremented when |
| --- | --- |
| `completed` | The member finishes a circle cleanly (deposit intact through completion) |
| `defaulted` | The member's deposit fails to cover a missed contribution and they are marked defaulted |

That is the entire record. There is no score, no weighting, no decay. The raw counts are public and anyone can read them for any address.

## Who can write it

Reputation is written **only** by authorized reporter contracts, never by users directly. The `circle` contract is registered as a reporter by the admin at deployment. When a circle completes or a member defaults, the `circle` contract calls the `reputation` contract to record the outcome.

Two checks gate every write:

1. Soroban invoker authorization — the call must come from the reporter contract itself.
2. An on-chain allow-list — the reporter must be marked `allowed = true` in the reputation contract's storage.

Both must pass. A user cannot inflate their own `completed` count or wipe a `defaulted` mark, because they cannot make the reputation contract accept a write.

## How it should be used

OpenAjo does not compute a credit score, and neither should apps built on it. The honest signal is the pair of counts:

* `completed: 3, defaulted: 0` — a reliable member across three circles.
* `completed: 1, defaulted: 2` — has walked away from two circles; a creator may want a higher deposit or may decline them.

The web app displays the raw counts and lets the group decide. Any scoring formula layered on top is a product choice, not a protocol fact — keep it out of the contract.

## Portability

Because reputation lives in its own contract keyed by Stellar address, it is not tied to any one circle or any one app. A member carries their record with them. A future version could let creators gate joining on a minimum completion count or a maximum default count; the data to support that already exists on-chain.
