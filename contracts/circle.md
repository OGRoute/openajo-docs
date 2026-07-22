# `circle`

The core contract. One deployed instance manages many circles and holds all funds. Source: [`openajo-contract/contracts/circle`](https://github.com/OGRoute/openajo-contract/tree/main/contracts/circle).

**Testnet:** [`CCLVOHGHDH32GWFAMCEMVHLNJSF6ENVHERYWU2OHUYWWLAOKLVR3HGKS`](https://stellar.expert/explorer/testnet/contract/CCLVOHGHDH32GWFAMCEMVHLNJSF6ENVHERYWU2OHUYWWLAOKLVR3HGKS)

## Types

```rust
pub enum CircleStatus { Open, Active, Completed, Cancelled }

pub struct Circle {
    creator: Address,
    token: Address,        // SAC address
    contribution: i128,    // per member per cycle, raw units
    deposit: i128,         // flat security bond
    size: u32,             // members required to activate
    period_secs: u64,      // seconds per cycle
    status: CircleStatus,
    started_at: u64,       // 0 until Active
    current_cycle: u32,    // 0-based
}

pub struct MemberState {
    deposit_remaining: i128,
    received: bool,
    defaulted: bool,
}
```

All amounts are `i128` in the token's raw units (USDC and XLM SACs use 7 decimals). There is no floating-point math in the contract.

## State-changing functions

### `initialize(admin: Address, reputation: Address)`

Callable once. Stores the admin and the address of the [`reputation`](reputation.md) contract. Panics `AlreadyInitialized` if called again.

### `create_circle(creator, token, contribution, deposit, size, period_secs) -> u32`

* **Auth:** `creator`
* Validates `contribution > 0`, `deposit >= 0`, `size >= 2`, `period_secs >= 3600` (else `BadParams`).
* Transfers `deposit` from `creator` into the contract.
* Creates the circle in `Open` status with `creator` as the first member.
* Returns the new circle id (sequential from 0).

### `join(circle_id: u32, member: Address)`

* **Auth:** `member`
* Requires status `Open`, `member` not already in the circle, and room remaining (else `BadStatus` / `AlreadyMember` / `CircleFull`).
* Transfers `deposit` from `member` into the contract.
* If the circle is now full, sets status to `Active`, records `started_at`, and emits `start`.

### `leave(circle_id: u32, member: Address)`

* **Auth:** `member`
* Only while `Open`, and not the creator (else `BadStatus` / `IsCreator`).
* Refunds the member's deposit and removes them.

### `cancel(circle_id: u32, creator: Address)`

* **Auth:** `creator` (must match the stored creator, else `NotCreator`)
* Only while `Open` (else `BadStatus`).
* Refunds every member's deposit and sets status to `Cancelled`.

### `contribute(circle_id: u32, member: Address)`

* **Auth:** `member`
* Requires status `Active`, `member` not defaulted, and no contribution yet this cycle (else `BadStatus` / `Defaulted` / `AlreadyPaid`).
* Transfers `contribution` from `member` into the contract and marks them paid for the current cycle.

### `settle_cycle(circle_id: u32)`

* **Auth:** none — **permissionless.**
* Requires status `Active` and the cycle to be due: either every active member has paid, or the deadline has passed (else `BadStatus` / `NotDue`).
* Slashes missed contributions from deposits, marks and reports any members whose deposit is exhausted as defaulted, pays the pot to the next member in join order, and either advances the cycle or completes the circle. See [Deposits & slashing](../protocol/deposits-and-slashing.md) for the exact procedure.

## Read-only functions

| Function | Returns |
| --- | --- |
| `get_circle(circle_id: u32)` | `Circle` (panics `NotFound` if absent) |
| `get_members(circle_id: u32)` | `Vec<Address>` in join order |
| `get_member(circle_id: u32, member: Address)` | `MemberState` (panics `NotMember` if absent) |
| `cycle_deadline(circle_id: u32)` | `u64` = `started_at + (current_cycle + 1) × period_secs` |
| `total_circles()` | `u32` — total circles ever created |

## Errors

| Code | Name | Meaning |
| --- | --- | --- |
| 1 | `NotInitialized` | Contract not initialized |
| 2 | `AlreadyInitialized` | `initialize` called twice |
| 3 | `NotFound` | No circle with that id |
| 4 | `BadStatus` | Action not allowed in the current status |
| 5 | `AlreadyMember` | Address already in the circle |
| 6 | `NotMember` | Address not in the circle |
| 7 | `CircleFull` | Circle already at `size` |
| 8 | `AlreadyPaid` | Already contributed this cycle |
| 9 | `Defaulted` | Membership has defaulted |
| 10 | `NotDue` | Cycle not due to settle yet |
| 11 | `IsCreator` | Creator cannot leave — cancel instead |
| 12 | `BadParams` | Invalid create parameters |
| 13 | `NotCreator` | Only the creator can do that |
