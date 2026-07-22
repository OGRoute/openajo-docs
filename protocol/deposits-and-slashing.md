# Deposits & slashing

This is the mechanism that answers the oldest question in rotating savings: **what stops someone who has already collected from walking away?**

## The security deposit

Every member escrows a flat `deposit` when they join, on top of their per-cycle contributions. The deposit is held by the contract and returned at completion — unless it is used to cover a missed contribution.

The deposit **prices** default. It does not fully collateralize a member. Requiring a deposit equal to the whole obligation (`size × contribution`) would defeat the purpose of an _ajo_ — the point is to get a lump sum before you have saved the whole amount. So the deposit is set by the creator at a level that makes missing a payment costly, while [reputation](reputation.md) makes repeat defaulting visible across circles.

## Slashing at settlement

When `settle_cycle` runs, for each active member who did **not** contribute this cycle:

1. The contract slashes `min(deposit_remaining, contribution)` from their deposit into the pot.
2. If that slash is **less than** a full `contribution`, the member has run out of deposit. They are marked **defaulted**: skipped in all future rotation, blocked from contributing, and reported to the reputation registry.

The pot for the cycle is therefore:

```
pot = contribution × (members who paid) + slashes from members who missed
```

The recipient still receives a full pot as long as deposits cover the misses.

## Worked example

Three members — **A**, **B**, **C** — each with `contribution = 100`, `deposit = 150`. Payout order is A, B, C.

### Cycle 0 — C misses, deposit covers it

A and B pay 100 each. C pays nothing.

* C is slashed `min(150, 100) = 100`. C's deposit drops to **50**. The slash (100) is a full contribution, so C is **not** defaulted yet.
* Pot = `100 (A) + 100 (B) + 100 (slash from C) = 300`.
* A receives **300**.

### Cycle 1 — C misses again, deposit runs out

A and B pay 100 each. C pays nothing.

* C is slashed `min(50, 100) = 50`. C's deposit drops to **0**. The slash (50) is less than a full contribution, so C is now **defaulted** and reported.
* Pot = `100 (A) + 100 (B) + 50 (slash from C) = 250`.
* B receives **250**.

### Completion

A and B have both received. C is defaulted and will never receive. No active member is still owed a payout, so the circle **completes**. A and B get their 150 deposits back; C's deposit is gone.

Final positions (verified on-chain in the test suite):

| Member | Deposits | Contributions | Received | Deposit refund | Net |
| --- | --- | --- | --- | --- | --- |
| A | −150 | −200 | +300 | +150 | **+100** |
| B | −150 | −200 | +250 | +150 | **+50** |
| C | −150 | 0 | 0 | 0 | **−150** |

The three positions sum to zero. C's −150 is exactly the deposit and it flowed to A and B as slashes. The contract ends holding nothing.

## Design consequences

* A member who defaults **after** already receiving their pot loses their deposit but keeps the pot — the deposit is what they forfeit, and the reputation ding is what follows them.
* Because default is priced rather than forbidden, circles can complete even when someone drops out mid-way — the group is compensated from the deposit, and rotation skips the defaulter.
* Setting a higher `deposit` makes a circle safer for members but raises the barrier to join. That trade-off is the creator's to make per circle.
