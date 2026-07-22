# Local setup

You do not need every toolchain. Pick by what you are working on.

| Working on | You need |
| --- | --- |
| Web app, SDK, API routes | **Node 20+** only |
| The Soroban contracts | Node + **Rust** + `wasm32v1-none` + **Stellar CLI** |
| The indexer | Node + **Postgres** |

## Web app and SDK

```bash
git clone https://github.com/OGRoute/openajo-app
cd openajo-app
npm install
cp .env.example apps/web/.env.local     # testnet contract IDs are pre-filled
npm run dev --workspace apps/web        # http://localhost:3000
```

That is enough to create, join, contribute to, and settle real circles against the deployed testnet contracts. The app degrades gracefully without the indexer — live chain reads still work; lists and stats show as unavailable.

Run the SDK tests:

```bash
npm test --workspace packages/sdk
```

## Indexer

Needs a Postgres database.

```bash
cd indexer
cp ../.env.example .env     # set DATABASE_URL and a recent START_LEDGER
npx prisma migrate deploy
npm run dev                 # http://localhost:8080/health
```

Set `START_LEDGER` to a ledger within RPC's ~7-day event retention window, otherwise the first poll returns an error. Leave `CRANK_SECRET` unset unless you want this instance auto-settling due circles.

## Contracts

```bash
git clone https://github.com/OGRoute/openajo-contract
cd openajo-contract
rustup target add wasm32v1-none
cargo test                                    # 19 tests
cargo build --target wasm32v1-none --release
```

Deploy your own instance to testnet (creates and funds an identity, deploys both contracts in dependency order, wires the reporter, prints the IDs):

```bash
./scripts/deploy.sh
```

Put the printed IDs into `apps/web/.env.local` and `indexer/.env`.

### Two toolchain traps

**Build target must be `wasm32v1-none`.** Current Rust emits post-MVP WebAssembly features on `wasm32-unknown-unknown` that the Soroban VM rejects at upload with `reference-types not enabled`. The workspace pins the correct target in `rust-toolchain.toml`.

**`ed25519-dalek` version conflict.** If `cargo test` fails compiling `soroban-env-host` with a `ChaCha20Rng: CryptoRng` trait error, pin it:

```bash
cargo update -p ed25519-dalek@3.0.0 --precise 2.2.0
```

The committed `Cargo.lock` already has this; you only need it if you regenerate the lockfile.

## Environment variables

| Variable | Used by | Notes |
| --- | --- | --- |
| `NEXT_PUBLIC_RPC_URL` | web | `https://soroban-testnet.stellar.org` |
| `NEXT_PUBLIC_NETWORK_PASSPHRASE` | web | `Test SDF Network ; September 2015` |
| `NEXT_PUBLIC_CIRCLE_CONTRACT_ID` | web | Deployed `circle` ID |
| `NEXT_PUBLIC_REPUTATION_CONTRACT_ID` | web | Deployed `reputation` ID |
| `NEXT_PUBLIC_TOKEN_ID` | web | SAC address; derive with `stellar contract id asset --asset native --network testnet` |
| `NEXT_PUBLIC_READ_SOURCE` | web | Any funded account ID; used as the source for read-only simulations. No secret. |
| `NEXT_PUBLIC_INDEXER_URL` | web | Indexer base URL |
| `RPC_URL`, `NETWORK_PASSPHRASE`, `CIRCLE_CONTRACT_ID`, `REPUTATION_CONTRACT_ID`, `READ_SOURCE` | indexer | Same values, server-side |
| `DATABASE_URL` | indexer | Postgres connection string |
| `START_LEDGER` | indexer | Back-fill start; must be within RPC retention |
| `CRANK_SECRET` | indexer | Optional `S…`; enables auto-settlement |
| `PORT` | indexer | Default `8080` |

Never commit a secret. `.env*` files are gitignored; `.env.example` holds placeholders and public testnet values only.
