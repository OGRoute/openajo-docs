---
description: Rotating savings on Stellar, enforced by a smart contract instead of a collector you have to trust.
---

# Introduction

**OpenAjo** brings Nigeria's rotating savings groups — _ajo_, _esusu_, _adashe_ — on-chain. A group agrees to contribute a fixed amount on a schedule; each cycle, the whole pot pays out to the next member in turn. OpenAjo replaces the trusted human collector with a [Soroban](https://developers.stellar.org/docs/build/smart-contracts/overview) smart contract on Stellar that no one controls.

## The problem

Rotating savings is how a large share of Nigeria saves. Per [EFInA](https://efina.org.ng/research/)'s Access to Financial Services survey, **14.6 million Nigerian adults relied on informal mechanisms only in 2018** — savings groups, _esusu_, _ajo_, and collectors — up from 9.4 million in 2016. These groups work on trust, and that trust is the weak point:

* The collector holds everyone's money between cycles. "The _ajo_ collector ran away with our money" is a loss story millions of people know first-hand.
* Records are on paper or in one person's head. Disputes have no source of truth.
* A member who collects their payout early has no reason to keep contributing.

Existing apps — AjoMoney, Thrifto, Alajo, KoloPay — digitize the ledger, but they replace the collector with a **company** you must trust instead. The custodian changes; the custody problem doesn't.

## How OpenAjo is different

There is no custodian. A Soroban contract holds the funds and enforces the rules:

* **Escrow** — contributions and deposits are locked in the contract, not in anyone's account.
* **Enforced order** — payout order is fixed at activation and executed by code.
* **Priced default** — every member posts a security deposit. Miss a contribution and the shortfall is slashed from your deposit into the pot, so the group is made whole.
* **Portable reputation** — a member who defaults is recorded permanently in an on-chain registry, visible to every future circle.
* **No trusted operator** — anyone can trigger a due settlement (`settle_cycle` is permissionless), so circles keep moving without a middleman.

## Why Stellar

* **Fees are fractions of a cent.** A contribution can be as small as ₦1,000-worth of USDC without fees eating the savings — the difference between viable and not for this use case.
* **USDC has real on/off-ramps in Nigeria**, so the value in a circle is spendable, not stuck.
* **Soroban** gives the contract the escrow, authorization, and cross-contract calls the protocol needs.

## Live on testnet

The contracts are deployed and verified on the Stellar test network.

| Contract | ID |
| --- | --- |
| `circle` | [`CCLVOHGHDH32GWFAMCEMVHLNJSF6ENVHERYWU2OHUYWWLAOKLVR3HGKS`](https://stellar.expert/explorer/testnet/contract/CCLVOHGHDH32GWFAMCEMVHLNJSF6ENVHERYWU2OHUYWWLAOKLVR3HGKS) |
| `reputation` | [`CDXPH2PYUTRW7GV57X6CJH3E3JOPROSC23NXPMAXOO3EOBI5UTCB2GTQ`](https://stellar.expert/explorer/testnet/contract/CDXPH2PYUTRW7GV57X6CJH3E3JOPROSC23NXPMAXOO3EOBI5UTCB2GTQ) |

## Where to go next

* New to the protocol? Read [Circle lifecycle](protocol/lifecycle.md).
* Want to use it? Start with [Set up a wallet](guides/wallet.md).
* Building on it? See [Architecture](developers/architecture.md) and the [SDK reference](developers/sdk.md).

> OpenAjo is unaudited and deployed to testnet only. Do not use it with mainnet funds.
