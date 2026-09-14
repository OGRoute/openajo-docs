# Contributing

OpenAjo is MIT-licensed and open to contributions. Issues are scoped by maintainers and labelled by the toolchain they need, so you can contribute to the web app without ever installing Rust.

## Repositories

| Repo | What lives there |
| --- | --- |
| [openajo-contract](https://github.com/OGRoute/openajo-contract) | Soroban contracts (Rust), tests, deploy script |
| [openajo-app](https://github.com/OGRoute/openajo-app) | SDK, indexer, web app |
| [openajo-docs](https://github.com/OGRoute/openajo-docs) | This documentation |

## Picking work

Comment on an issue to claim it before starting. Labels tell you what you are getting into:

* `good first issue` — self-contained, clear acceptance criteria
* `help wanted` — larger but well-specified
* `needs design` — discuss the approach in the issue **before** writing code

## Before opening a pull request

**App repo:**

```bash
npm run typecheck
npm test --workspace packages/sdk
npm run build --workspace apps/web
```

**Contract repo:**

```bash
cargo fmt --all
cargo clippy --all-targets -- -D warnings
cargo test
cargo build --target wasm32v1-none --release
```

CI enforces all of these plus secret scanning. A PR that fails CI will not be reviewed until it is green.

## Standards

**Contracts**

* `#![no_std]`; no `unwrap()` or `expect()` outside tests; no floating-point.
* All money is `i128` in raw token units.
* Every user action calls `require_auth()`; every public function documents who may call it.
* Every persistent storage write extends TTL.
* Every new function ships with tests, including `#[should_panic]` error paths. Balance-changing logic asserts **exact** balances.

**App**

* TypeScript `strict`. No `any` except at a documented library boundary.
* `bigint` for amounts end-to-end; format only at display, parse only at input.
* No server-held user keys. Users sign with Freighter.
* UI must work in light and dark themes and be keyboard-accessible.

## Cross-repo changes

Contract event shapes are a compatibility contract with the indexer. Changing a topic or data tuple breaks every consumer, so it requires **coordinated issues in both repos** with an explicit "Depends on" reference, and the contract change must ship and deploy first.

## Commits

Conventional format: `type(scope): description`.

* Contract scopes: `circle`, `reputation`, `ci`, `docs`
* App scopes: `sdk`, `indexer`, `web`, `ci`, `docs`

One logical unit per commit.

## Code of conduct

Everyone taking part in OpenAjo repositories is expected to follow the [Code of Conduct](https://github.com/OGRoute/openajo-docs/blob/main/CODE_OF_CONDUCT.md), based on the Contributor Covenant 2.1.

## Security

Do not open public issues for vulnerabilities. Use GitHub's private vulnerability reporting on the affected repository. OpenAjo escrows funds — auth bypasses, settlement manipulation, and anything that strands or leaks escrowed value are the highest-priority findings.

OpenAjo is **unaudited and testnet-only**. Do not deploy it with mainnet funds.
