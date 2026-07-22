# Join & contribute

## Joining

Browse **Circles** and open one with status **Open**. Before you join, check:

* **Contribution and cycle length** — can you pay this reliably, every cycle, for the full run?
* **Your position** — members are paid in join order. If you are joining a 6-person circle as the 5th member, you collect 5th.
* **Total commitment** — a 6-person circle means 6 cycles. You pay `6 × contribution` overall and collect one pot.
* **The deposit** — you post this now and get it back at completion, unless it is slashed.
* **The other members** — click any address to see their on-chain [reputation](../protocol/reputation.md).

Click **Join**, approve in Freighter, and your deposit is escrowed. If you change your mind before the circle fills, you can **Leave** and take your deposit back. Once the circle activates, leaving is no longer possible.

## Contributing each cycle

When the circle is Active, open it and click **Contribute**. This transfers one contribution to the contract and marks you paid for the current cycle.

The circle page shows the **deadline** for the current cycle. Pay before it.

You cannot contribute twice in one cycle, and you cannot contribute after the circle completes.

## If you miss a deadline

Nothing is forgiven, but nothing is catastrophic either — the deposit absorbs it:

1. At settlement, the contract slashes one contribution from your deposit and puts it in the pot. The group is made whole; the recipient still gets a full pot.
2. Your `deposit_remaining` drops. The circle page shows it.
3. If your deposit cannot cover a full contribution, you are marked **defaulted**.

## What defaulting means

* You are **skipped in the rotation permanently** — if you had not yet collected, you never will.
* You **cannot contribute** any more.
* Your remaining deposit is gone.
* A default is recorded against your address in the reputation contract, visible to anyone considering a circle with you in future.

If you had already collected your pot before defaulting, you keep the pot but lose the deposit and take the reputation hit. That asymmetry is why the deposit should not be trivial.

## Getting your deposit back

Your deposit is returned automatically when the circle **completes** — no action needed. It arrives in the same transaction that finishes the circle, along with a completion recorded to your reputation.
