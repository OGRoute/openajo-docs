# Payouts & completion

## How a cycle settles

A cycle is ready to settle when **either**:

* every active member has contributed, **or**
* the cycle deadline has passed.

Then anyone clicks **Settle cycle**. There is no admin — the button is available to any member, and the OpenAjo indexer also runs an automatic settler, so cycles advance even if nobody presses anything.

Settlement does three things in one transaction:

1. **Collects the pot** — contributions already paid in, plus a slash from the deposit of anyone who missed.
2. **Pays the next member in join order** who has not yet received.
3. **Advances the cycle**, or completes the circle if everyone active has been paid.

## Receiving your pot

You do not claim a payout — it is pushed to your wallet by the settlement transaction when your turn comes. Watch for it on the circle's timeline as a `payout` entry, or check the transaction on [Stellar Expert](https://stellar.expert/explorer/testnet).

The amount is the full pot for that cycle. In a clean circle of `N` members that is `N × contribution`.

## Why settlement can happen without you

`settle_cycle` requires no authorization by design. The contract can only move money according to its own rules, so letting anyone trigger it removes the need for a trusted operator — nobody has to be online, trusted, or paid for the circle to keep running. A settler pays only the network fee and cannot redirect a single unit of the pot.

## Completion

The circle completes as soon as **no active member is still owed a payout**. Usually that is after `N` cycles. It can happen sooner if members default — a defaulted member is skipped, so the circle finishes once everyone who is still active has collected.

On completion, in the same transaction:

* Every non-defaulted member's remaining deposit is refunded.
* A `completed` is recorded for each of them in the reputation contract.
* The circle's status becomes **Completed** and it stops accepting calls.

## Reading the timeline

Each circle page shows a full history built from on-chain events:

| Entry | Meaning |
| --- | --- |
| `join` | Someone joined and posted a deposit |
| `start` | The circle filled and activated |
| `contrib` | A member paid a cycle |
| `slash` | A missed contribution was taken from a deposit |
| `default` | A member's deposit ran out; they are now skipped |
| `payout` | The pot was paid to a member |
| `complete` | The circle finished and deposits were refunded |

Every row corresponds to a real Stellar transaction you can verify on a block explorer.
