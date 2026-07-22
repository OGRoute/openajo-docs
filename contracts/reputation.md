# `reputation`

Permanent, cross-circle record of member outcomes. Written only by authorized reporter contracts; read by anyone. Source: [`openajo-contract/contracts/reputation`](https://github.com/OGRoute/openajo-contract/tree/main/contracts/reputation).

**Testnet:** [`CDXPH2PYUTRW7GV57X6CJH3E3JOPROSC23NXPMAXOO3EOBI5UTCB2GTQ`](https://stellar.expert/explorer/testnet/contract/CDXPH2PYUTRW7GV57X6CJH3E3JOPROSC23NXPMAXOO3EOBI5UTCB2GTQ)

## Types

```rust
pub struct Reputation {
    completed: u32,   // circles finished cleanly
    defaulted: u32,   // circles defaulted on
}
```

## Functions

### `initialize(admin: Address)`

Callable once. Stores the admin who manages the reporter allow-list. Panics `AlreadyInitialized` if called again.

### `set_reporter(admin: Address, reporter: Address, allowed: bool)`

* **Auth:** `admin` (must match the stored admin, else `NotAdmin`)
* Adds or removes `reporter` from the allow-list. The `circle` contract's address is registered here at deployment.

### `report_completion(reporter: Address, member: Address)`

* **Auth:** `reporter`, and `reporter` must be allow-listed (else `NotReporter`)
* Increments `member.completed`. Emits `("rep", "complete")`.

### `report_default(reporter: Address, member: Address)`

* **Auth:** `reporter`, and `reporter` must be allow-listed (else `NotReporter`)
* Increments `member.defaulted`. Emits `("rep", "default")`.

### `get_reputation(member: Address) -> Reputation`

* **Auth:** none
* Returns the member's counts, or `{ completed: 0, defaulted: 0 }` for an unknown address.

## Errors

| Code | Name | Meaning |
| --- | --- | --- |
| 1 | `NotInitialized` | Contract not initialized |
| 2 | `AlreadyInitialized` | `initialize` called twice |
| 3 | `NotAdmin` | Caller is not the stored admin |
| 4 | `NotReporter` | Caller is not an allow-listed reporter |

## Trust model

The two-check gate — invoker authorization **and** the storage allow-list — means a user can never write their own reputation. Only a contract the admin has explicitly authorized (the `circle` contract) can record outcomes, and only as a side effect of a real completion or default it just processed.
