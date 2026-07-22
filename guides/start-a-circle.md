# Start a circle

You start a circle, choose the rules, and become its first member. Everyone who joins accepts those rules — they cannot be changed afterwards.

## Before you start

You need a [funded wallet on testnet](wallet.md) and a rough plan: how much each person can pay per cycle, how often, and how many people.

## The five choices

**Contribution per cycle.** What each member pays every cycle. If five people each pay 10 USDC weekly, the weekly pot is 50 USDC.

**Security deposit.** What each member escrows up front. This is the money slashed if someone misses a payment. Higher deposit = safer for the group, harder to join. A deposit around one to two contributions is a reasonable starting point.

**Members.** How many people the circle needs before it activates. This is also how many cycles it runs — five members means five cycles, and each person collects once.

**Cycle length.** Daily, weekly, or monthly. Match it to how people actually earn.

**Token.** The Stellar asset used. On testnet the app uses native XLM; a production deployment would use USDC.

## Creating it

1. Open the app and click **Start a circle**.
2. Fill in the five values.
3. Click **Create circle** and approve the transaction in Freighter.

Your deposit is transferred to the contract in that same transaction. The circle is now **Open** and you are member 1.

## Filling it

Share the circle link. Each person joins by posting the same deposit. When the last member joins, the circle **activates automatically** in that transaction — the payout order is locked to join order, and cycle 1 begins immediately.

You are position 1, so you collect the first pot.

## If it doesn't fill

While the circle is Open you can **cancel** it. Every member — including you — gets their full deposit back and the circle closes. There is no penalty and no fee beyond network costs.

## Choosing well

* **Payout order is join order.** Being first means collecting before you have paid much in — that is the advantage, and it is also why the deposit and reputation systems exist. Members who join later carry more risk, so keep the group to people who have reason to trust each other or who have a visible [reputation](../protocol/reputation.md).
* **Size is commitment length.** A ten-person monthly circle runs ten months. Longer circles mean more chances for someone's situation to change.
* **Set the deposit to the pain you want a default to cause.** A trivial deposit makes walking away cheap.
