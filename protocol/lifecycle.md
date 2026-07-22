# Circle lifecycle

A circle moves through four states. Every transition is a contract call, and the contract holds all funds from the first deposit to the last refund.

```
Open ──(fills to size)──▶ Active ──(all cycles settled)──▶ Completed
  │
  └──(creator cancels)──▶ Cancelled
```

## Open

A circle is created with five parameters, fixed for its lifetime:

| Parameter | Meaning | Constraint |
| --- | --- | --- |
| `token` | The Stellar asset used (a SAC address; USDC in production) | — |
| `contribution` | Amount each member pays per cycle | `> 0` |
| `deposit` | Flat security bond each member escrows on joining | `>= 0` |
| `size` | Number of members required to activate | `>= 2` |
| `period_secs` | Length of one cycle in seconds | `>= 3600` |

The creator is the first member and posts the deposit when creating the circle. Others join, each posting the same deposit. While Open:

* A non-creator member may **leave** and reclaim their deposit.
* The creator may **cancel**, refunding every member.

## Active

When the number of members reaches `size`, the circle activates automatically on the joining transaction. At that moment:

* `started_at` is set to the current ledger timestamp.
* Payout order is frozen to **join order** — the creator is position 1.
* Cycle 0 begins.

Each cycle:

1. Every active member calls `contribute` to pay `contribution` into the contract.
2. Once all active members have contributed **or** the cycle deadline has passed, anyone calls `settle_cycle`.
3. Settlement pays the pot to the next member in join order who has not yet received, then either advances to the next cycle or completes the circle.

The deadline for the current cycle is `started_at + (current_cycle + 1) × period_secs`.

### Permissionless settlement

`settle_cycle` takes no authorization. Funds can only move according to the protocol rules encoded in the contract, so it is safe for anyone — a member, a bot, or the OpenAjo indexer — to trigger a due cycle. This is what removes the need for a trusted operator.

## Completed

A circle completes when no active member is still owed a payout. On completion:

* Every non-defaulted member's remaining deposit is refunded.
* A completion is recorded for each of them in the [reputation](reputation.md) registry.

In the clean case where a circle of `N` members runs `N` cycles with everyone paying, every member pays `N × contribution` in total and receives one pot of `N × contribution`, then gets their deposit back. Net position: zero, minus network fees. The value of a circle is not yield — it is **access to a lump sum sooner than saving alone would allow**, backed by the discipline of the group.

## Cancelled

If the creator cancels while the circle is still Open, all deposits are returned and the circle is closed permanently. A circle cannot be cancelled once Active — at that point funds are in motion and the settlement rules govern.

## What the contract guarantees

* No party, including the creator, can withdraw another member's funds.
* Payout order cannot be changed after activation.
* A member cannot receive twice.
* Contributions and deposits are only ever paid out as pot payouts or refunds defined by these rules.
