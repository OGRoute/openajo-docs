# Glossary

This glossary explains common OpenAjo and Stellar terms used throughout the documentation.

## Adashe

A traditional rotating savings group commonly used in northern Nigeria where members contribute regularly and take turns receiving the pooled money. See the [Introduction](README.md).

## Ajo

A rotating savings group where members contribute a fixed amount on a schedule and each member receives the pooled funds in turn. See the [Introduction](README.md).

## Contract ID

A unique identifier for a deployed smart contract on Stellar. OpenAjo uses contract IDs to identify its Circle and Reputation contracts. See the [Introduction](README.md).

## Contribution

The fixed amount each member pays during every cycle of a circle. See [Join & contribute](guides/join-and-contribute.md).

## Crank

An automated process that submits settlement transactions so circles continue progressing even when no participant manually settles them. See [Architecture](developers/architecture.md).

## Cycle

One contribution period in a circle. During each cycle, members contribute funds and one member receives the payout. See [Circle lifecycle](protocol/lifecycle.md).

## Default

A member is considered in default when their remaining deposit cannot fully cover a missed contribution. See [Deposits & slashing](protocol/deposits-and-slashing.md).

## Deposit

A security amount each member locks when joining a circle. It helps protect the group if someone misses a contribution. See [Deposits & slashing](protocol/deposits-and-slashing.md).

## Esusu

A traditional rotating savings group used in many West African communities. It follows the same basic savings model as an ajo. See the [Introduction](README.md).

## Freighter

A Stellar wallet used to sign transactions and interact with OpenAjo. See [Set up a wallet](guides/wallet.md).

## Network passphrase

A value that identifies which Stellar network an application connects to, such as testnet. See [Local setup](developers/local-setup.md).

## Payout

The pooled funds paid to the member whose turn it is during a cycle. See [Payouts & completion](guides/payouts-and-completion.md).

## Reputation

A permanent on-chain record showing how reliably a member has completed or defaulted in previous circles. See [Reputation](protocol/reputation.md).

## ROSCA

Short for **Rotating Savings and Credit Association**, the general financial term for savings groups such as ajo, esusu, and adashe. See the [Introduction](README.md).

## Rotation

The order in which members receive payouts during a circle. See [Circle lifecycle](protocol/lifecycle.md).

## SAC

Short for **Stellar Asset Contract**, the standard smart contract representation of an asset on Soroban. See [Circle lifecycle](protocol/lifecycle.md).

## Settlement

The transaction that processes a completed cycle, distributes the payout, applies any required slashing, and advances the circle. See [Payouts & completion](guides/payouts-and-completion.md).

## Slashing

The process of taking funds from a member's deposit to cover a missed contribution. See [Deposits & slashing](protocol/deposits-and-slashing.md).

## Soroban

The smart contract platform on Stellar that runs the OpenAjo contracts. See the [Introduction](README.md).

## Stroop

The smallest unit of XLM. One XLM equals 10,000,000 stroops. See [Set up a wallet](guides/wallet.md).

## Testnet

A public Stellar network used for development and testing without real funds. See [Set up a wallet](guides/wallet.md).

## XLM

The native asset of the Stellar network. OpenAjo uses test XLM while running on testnet. See [Set up a wallet](guides/wallet.md).