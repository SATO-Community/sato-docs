# SatoHook Protocol Contract

## Overview

`SatoHook` is the execution engine of the SATO protocol.

While the SATO ERC-20 contract maintains token balances and supply accounting, the hook is responsible for enforcing the protocol's monetary rules.

Every protocol buy and sell passes through the hook before settlement.

Its responsibilities include:

* Bonding curve execution
* Buy validation
* Sell validation
* Reserve accounting
* Protocol fee accounting
* Entropy adjustment
* Cooldown enforcement
* Mint authorization
* Burn authorization
* Pool validation

The hook therefore acts as the protocol layer, while the ERC-20 contract serves as the accounting layer.

---

# Design Philosophy

SatoHook is designed around deterministic protocol execution.

Rather than relying on discretionary governance or privileged operators, protocol behavior is defined directly in immutable smart contract logic.

The hook does not make subjective decisions.

Instead, it validates protocol rules and executes deterministic state transitions.

This separation allows the protocol to distinguish between:

* **Token accounting** (handled by `SatoToken`)
* **Monetary execution** (handled by `SatoHook`)
* **Curve mathematics** (handled by `Curve`)

Each component has a narrowly defined responsibility.

---

# Protocol Lifecycle

The protocol follows a deterministic lifecycle.

```text
                Deploy
                   │
                   ▼
          Pool Initialization
                   │
                   ▼
          One-Time Minter Lock
                   │
                   ▼
            Protocol Active
             ┌──────────────┐
             │              │
             ▼              ▼
         Buy Execution   Sell Execution
             │              │
             └──────┬───────┘
                    ▼
            Reserve Updates
                    │
                    ▼
          Curve Exhaustion
                    │
                    ▼
          Self Deprecation
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
  Reject New Buys      Continue Sells
```

Once deployed, the protocol operates without administrative intervention.

---

# System Architecture

```text
                    User
                      │
                      ▼
          Uniswap v4 Router
                      │
                      ▼
      Uniswap v4 PoolManager
                      │
                      ▼
                 SatoHook
          ┌─────────┴─────────┐
          ▼                   ▼
   Curve Library         SatoToken
          │                   │
          └─────────┬─────────┘
                    ▼
             ERC-20 State Update
```

The PoolManager provides settlement.

The Curve library provides deterministic pricing mathematics.

The ERC-20 contract records balances.

SatoHook coordinates protocol execution.

---

# Core Responsibilities

The hook performs the following protocol responsibilities:

* Validate the intended pool.
* Reject unsupported pool configurations.
* Prevent external liquidity additions.
* Process protocol buys.
* Process protocol sells.
* Compute fair curve issuance.
* Compute inverse curve redemption.
* Maintain cumulative curve state.
* Track protocol fees.
* Track reserve accounting.
* Enforce cooldown rules.
* Apply entropy adjustments.
* Trigger ERC-20 mint operations.
* Trigger ERC-20 burn operations.

The hook intentionally does **not** perform token accounting directly.

Instead, all balance changes are delegated to the ERC-20 contract.

---

# Immutable Configuration

Several protocol parameters are fixed at deployment.

These include:

| Parameter           | Description                      |
| ------------------- | -------------------------------- |
| Pool Manager        | Canonical Uniswap v4 PoolManager |
| Native Currency     | ETH                              |
| SATO Token          | ERC-20 accounting contract       |
| Pool Fee            | Fixed protocol fee tier          |
| Deployment Block    | Immutable deployment reference   |
| Previous Block Hash | Immutable genesis reference      |

These values cannot be modified after deployment.

---

# Protocol State

Unlike immutable configuration values, protocol state evolves over time.

Important state variables include:

| Variable          | Purpose                                                              |
| ----------------- | -------------------------------------------------------------------- |
| `ethCum`          | Cumulative ETH represented by the bonding curve after protocol fees. |
| `totalMintedFair` | Canonical curve supply used by the mathematical model.               |
| `feesAccrued`     | Total protocol fees accumulated by the hook.                         |
| `lastBuyBlock`    | Records the most recent buy block for each participant.              |
| `poolInitialized` | Indicates whether the intended pool has been initialized.            |
| `selfDeprecated`  | Indicates whether new buys have been permanently disabled.           |

These variables collectively represent the current monetary state of the protocol.

---

# Hook Permissions

SatoHook intentionally enables only the hook callbacks required for protocol execution.

| Hook Callback           | Purpose                                              |
| ----------------------- | ---------------------------------------------------- |
| `beforeInitialize`      | Validates the intended pool before activation.       |
| `beforeAddLiquidity`    | Rejects external liquidity additions.                |
| `beforeSwap`            | Executes protocol buy and sell logic.                |
| `beforeSwapReturnDelta` | Returns custom accounting deltas to the PoolManager. |

All remaining Uniswap v4 hook callbacks are disabled.

During deployment, the hook validates that its deployed address contains the correct permission bitmap.

This prevents accidental deployment with incorrect hook permissions.

---

# Pool Initialization

Before the protocol becomes active, the hook validates the pool configuration.

Initialization succeeds only if:

* the configured PoolManager is the expected instance,
* the pool uses native ETH,
* the second currency is the SATO ERC-20 token,
* the configured fee matches the protocol constant,
* the deployed hook matches the pool configuration.

If any validation fails, initialization reverts.

This guarantees that protocol execution cannot begin on an unintended pool configuration.

---

# Liquidity Model

Unlike conventional automated market makers, SATO does not accept externally supplied liquidity.

Any attempt to add liquidity through the pool triggers an immediate revert.

Instead of relying on liquidity providers, every trade is settled directly against the deterministic bonding curve.

The protocol therefore acts as the effective counterparty for both buys and sells.

Reserve accounting remains entirely under protocol control.

---

# Swap Entry Point

Every protocol trade enters through the `beforeSwap()` callback.

This callback is the primary execution entry point of the monetary protocol.

Before processing a trade, the hook validates:

* swap direction,
* swap type,
* protocol state,
* cooldown requirements,
* hook data,
* reserve availability.

Only after these validations succeed does execution continue to either the buy path or the sell path.

The next sections describe these execution paths in detail.

# Buy Execution

A protocol buy occurs when a user swaps native ETH for SATO.

The hook interprets this swap as a request to purchase newly issued tokens directly from the bonding curve.

The execution sequence is deterministic.

## Execution Pipeline

```text
ETH Input
    │
    ▼
Validate Buy
    │
    ▼
Calculate Protocol Fee
    │
    ▼
Update Curve Position
    │
    ▼
Curve.mintFor()
    │
    ▼
Apply Entropy Multiplier
    │
    ▼
Mint SATO
    │
    ▼
Update Reserve State
    │
    ▼
Record Buy Block
```

During execution the hook performs the following steps:

1. Verify that the protocol has not entered self-deprecation.
2. Reject oversized buy requests.
3. Calculate protocol fees.
4. Add the net ETH amount to the cumulative curve position.
5. Calculate fair token issuance using `Curve.mintFor()`.
6. Apply the entropy multiplier when applicable.
7. Mint SATO through the ERC-20 contract.
8. Update cumulative ETH (`ethCum`).
9. Update canonical fair supply (`totalMintedFair`).
10. Record the buyer's block number.
11. Accumulate protocol fees.
12. Return settlement deltas to the PoolManager.

No discretionary decisions occur during this process.

Every state transition follows deterministic protocol rules.

---

# Sell Execution

A protocol sell occurs when SATO is exchanged for native ETH.

Rather than trading against external liquidity providers, redemption is calculated directly from the inverse bonding curve.

## Execution Pipeline

```text
SATO Input
    │
    ▼
Cooldown Check
    │
    ▼
Convert To Fair Supply
    │
    ▼
Curve.burnFor()
    │
    ▼
Calculate Protocol Fee
    │
    ▼
Burn SATO
    │
    ▼
Transfer ETH
    │
    ▼
Update Reserve State
```

During execution the hook performs the following steps:

1. Verify cooldown requirements.
2. Convert entropy-adjusted supply into canonical fair supply.
3. Calculate ETH owed using `Curve.burnFor()`.
4. Calculate protocol fees.
5. Verify sufficient reserve backing.
6. Burn SATO through the ERC-20 contract.
7. Transfer ETH through PoolManager settlement.
8. Reduce cumulative ETH.
9. Reduce canonical fair supply.
10. Accumulate protocol fees.
11. Return settlement deltas.

Protocol sells always follow the deterministic inverse curve.

---

# Fair Supply Model

The protocol intentionally distinguishes between two supply measurements.

| Supply              | Purpose                                                |
| ------------------- | ------------------------------------------------------ |
| ERC-20 Total Supply | Actual number of SATO tokens currently in circulation. |
| `totalMintedFair`   | Canonical supply used by the bonding curve.            |

These values differ only because early protocol participants may receive entropy-adjusted issuance.

All pricing calculations use the canonical fair supply.

This guarantees that redemption pricing always follows the deterministic bonding curve regardless of temporary entropy adjustments.

---

# Entropy Mechanism

During the initial launch period the protocol introduces a bounded entropy adjustment.

The entropy multiplier applies only during the configured entropy window.

Its purpose is to introduce small, deterministic issuance variation during early participation.

Inputs include:

* Previous block hash
* Swapper address
* ETH input amount

The resulting multiplier remains bounded within a narrow interval.

```text
Approximately

0.90×
      to
1.10×
```

After the entropy window expires, protocol issuance follows the bonding curve without adjustment.

Entropy never changes the mathematical shape of the curve.

It only affects the amount received during the early launch phase.

---

# Cooldown Mechanism

The hook records the block number of every successful buy.

This value is stored in:

```
lastBuyBlock
```

Before a sell is processed, the hook verifies that the configured cooldown has elapsed.

The current implementation enforces a one-block delay.

This mechanism reduces same-block and immediate next-block buy/sell behavior.

Cooldown enforcement is deterministic and does not require administrative intervention.

---

# Deterministic Monetary Execution

Neither buy execution nor sell execution relies on privileged operators.

The hook validates protocol rules, computes deterministic monetary outcomes, and delegates token accounting to the ERC-20 contract.

All issuance and redemption therefore follow immutable protocol logic rather than discretionary human decisions.

# Reserve Accounting

The protocol maintains an internal accounting model for ETH reserves.

Rather than relying on external liquidity providers, reserve backing is derived from protocol execution.

Each successful buy increases the cumulative reserve represented by the bonding curve.

Each successful sell reduces the cumulative reserve according to the inverse curve.

The hook exposes a reserve view through:

```text
curveReserveEth()
```

The reported reserve is calculated as:

```text
Contract ETH Balance
        −
 Accrued Protocol Fees
```

This distinction separates protocol backing from accumulated fee revenue.

---

# Fee Accounting

Protocol fees are collected during both buy and sell execution.

Fees are calculated using the fixed protocol fee ratio defined by immutable constants.

Accumulated fees are tracked through:

```text
feesAccrued
```

Protocol fees are excluded from the reserve used to back token redemption.

This separation prevents fee revenue from being interpreted as reserve collateral.

---

# Self-Deprecation

The protocol contains an automatic self-deprecation mechanism.

When the canonical fair supply reaches the configured protocol threshold, the hook permanently disables new buys.

Once activated:

* New buy operations are rejected.
* Existing holders may continue selling.
* Reserve redemption remains available.
* No administrator action is required.

This transition is enforced entirely by immutable protocol logic.

Self-deprecation is irreversible.

---

# ETH Flow

ETH enters the protocol during successful buy execution.

```text
Buyer
   │
   ▼
PoolManager
   │
   ▼
SatoHook Reserve
```

ETH leaves the protocol during successful sell execution.

```text
SatoHook Reserve
        │
        ▼
PoolManager
        │
        ▼
Seller
```

The hook does not expose arbitrary ETH withdrawal functionality as part of normal protocol execution.

Reserve movements occur only through deterministic buy and sell settlement.

---

# Access Control

The protocol intentionally minimizes privileged execution.

Administrative permissions are intentionally absent.

Instead, protocol execution is restricted by execution context.

Key access restrictions include:

| Operation           | Restriction                      |
| ------------------- | -------------------------------- |
| Pool initialization | Expected pool configuration only |
| Buy execution       | PoolManager callback only        |
| Sell execution      | PoolManager callback only        |
| Mint                | Locked protocol minter only      |
| Burn                | Locked protocol minter only      |

The hook validates execution context before modifying protocol state.

---

# Security Model

SatoHook is designed around deterministic protocol execution rather than discretionary administration.

Important security properties include:

* Immutable deployment configuration.
* Validated hook permissions.
* Strict pool validation.
* No external liquidity additions.
* Deterministic bonding curve execution.
* Cooldown enforcement.
* Bounded entropy adjustment.
* Reserve-backed redemption.
* Locked ERC-20 mint authorization.
* Locked ERC-20 burn authorization.

The hook does not act as a discretionary administrator.

Instead, it executes immutable protocol rules.

---

# Relationship with SatoToken

The ERC-20 contract and the hook intentionally perform different responsibilities.

| Component | Responsibility             |
| --------- | -------------------------- |
| SatoToken | Token accounting           |
| SatoHook  | Monetary execution         |
| Curve     | Mathematical pricing model |

The hook never edits balances directly.

Instead, after validating protocol rules, it invokes the restricted `mint()` or `burn()` interface exposed by the ERC-20 contract.

This separation improves modularity, simplifies auditing, and isolates accounting from monetary logic.

---

# Scanner Interpretation

Automated scanners may detect that the hook can trigger mint and burn operations.

Viewed in isolation, this may resemble privileged token issuance.

Within the complete protocol architecture, however, minting and burning are not discretionary administrative actions.

They occur only after deterministic protocol validation performed by the hook.

Relevant architectural properties include:

* One-time minter locking.
* Immutable hook configuration.
* Deterministic bonding curve execution.
* Reserve-backed redemption.
* Absence of owner-controlled monetary policy.

Scanner results should therefore be interpreted together with the protocol architecture rather than by evaluating isolated functions.

Additional discussion is available in:

```text
docs/security/05_False_Positive_Analysis.md
```

---

# Summary

SatoHook is the execution engine of the SATO protocol.

It coordinates protocol buys and sells, executes bonding curve calculations, maintains reserve accounting, enforces cooldown and entropy rules, tracks protocol fees, and authorizes ERC-20 mint and burn operations.

Rather than relying on discretionary governance or administrative intervention, the hook executes immutable protocol rules through deterministic smart contract logic.

Together with the SatoToken accounting layer and the Curve mathematical library, SatoHook forms the core of the SATO monetary protocol.

---

## Related Documentation

* `03_ERC20.md`
* `05_Bonding_Curve.md`
* `06_Reserve_Model.md`
* `docs/security/01_Overview.md`
* `docs/security/05_False_Positive_Analysis.md`

---

**END OF FILE**

