# Architecture

OpenAjo is two repositories: the Soroban contracts, and the application layer that reads and writes them.

| Repo | Contents |
| --- | --- |
| [openajo-contract](https://github.com/OGRoute/openajo-contract) | Rust workspace: `circle` and `reputation` contracts, tests, deploy script |
| [openajo-app](https://github.com/OGRoute/openajo-app) | npm workspaces: `packages/sdk`, `indexer/`, `apps/web` |

## Topology

```
user ──▶ apps/web ──▶ indexer REST API  (lists, history, stats)
              │
              └─────▶ Soroban RPC       (writes signed in Freighter; live reads)

indexer ──▶ Soroban RPC getEvents ──▶ Postgres
indexer ──▶ settle_cycle (optional crank)
```

Two rules shape this design:

**Writes never touch a server.** The browser builds the transaction, Freighter signs it, and it goes straight to Soroban RPC. No OpenAjo server holds a user key or can move user funds. The only server-side key is the optional crank account, which pays fees to call the permissionless `settle_cycle` and can do nothing else.

**Reads come from whichever source is right for the job.** Live state that must be correct at the moment of action — a circle's status, your `MemberState`, the cycle deadline — is read directly from the contract by simulation. Lists, history, and aggregate stats come from the indexer, because scanning every circle over RPC on every page load would be slow and pointless.

## The indexer's folding strategy

Events tell the indexer **which** circles changed; they are not treated as the source of truth for state.

1. Poll `getEvents` for both contract IDs from the last stored ledger.
2. Store each raw event, keyed by the RPC event id so re-processing is idempotent.
3. For every circle touched, re-read `get_circle`, `get_members`, and each `get_member` by simulation and upsert the result.

This means the database cannot drift from the chain. If the indexer misses a window, falls behind, or is replayed, the next refresh reconciles it to on-chain truth. The alternative — applying event deltas to local state — accumulates error forever the first time a single event is missed.

> Soroban RPC retains roughly 7 days of events. `START_LEDGER` must be within retention. For a longer history, back-fill from an archive or accept that the indexer's timeline begins at first run; live state is always correct either way.

## The SDK boundary

`packages/sdk` is the only place that knows how to encode arguments to these contracts. Both the web app and the indexer import it, so the ScVal encoding, error decoding, and event decoding exist once.

Writes take a `SignFn` rather than a key:

```ts
export type SignFn = (xdrBase64: string, networkPassphrase: string) => Promise<string>;
```

The web supplies a Freighter-backed signer; the crank supplies a keypair signer. The transaction-building code is identical for both, and the SDK never sees a secret.

## Money handling

All amounts are `bigint` in raw token units end-to-end — contract (`i128`), SDK, indexer (Postgres `BigInt`), and API. There is no floating-point arithmetic anywhere in the stack. Values are formatted to a decimal string only at the point of display, and parsed back at the point of input. This is the standard defence against rounding errors silently changing balances.
