# SatoHook Protocol Contract

## Overview

`SatoHook` is the execution engine of the SATO protocol.

The SATO ERC-20 contract maintains balances, transfers, allowances, and total supply. `SatoHook` enforces the monetary rules that determine when tokens are minted, when tokens are burned, how ETH reserves are updated, and how buys and sells are settled.

In the protocol architecture:

* `SatoToken` is the accounting layer.
* `SatoHook` is the monetary execution layer.
* `Curve` is the mathematical pricing layer.
* Uniswap v4 `PoolManager` is the settlement layer.

Every protocol buy and sell is routed through the hook before final settlement.

---

## Design Philosophy

`SatoHook` is designed around deterministic protocol execution.

Rather than relying on discretionary governance, privileged operators, or externally supplied AMM liquidity, protocol behavior is encoded directly in smart contract logic.

The hook does not decide policy subjectively.

It validates a transaction against protocol rules, executes deterministic state transitions, and delegates token accounting to `SatoToken`.

The result is a modular architecture where each component has a specific role.

| Component     | Role                      |
| ------------- | ------------------------- |
| `SatoToken`   | ERC-20 accounting         |
| `SatoHook`    | Protocol execution        |
| `Curve`       | Bonding curve mathematics |
| `PoolManager` | Uniswap v4 settlement     |

---

## Protocol Lifecycle

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
          Self-Deprecation
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
  Reject New Buys      Continue Sells
```

After deployment and initialization, normal protocol operation does not require discretionary administrative intervention.

---

## System Architecture

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

The hook provides protocol execution.

The Curve library provides deterministic pricing calculations.

The ERC-20 contract records token state updates.

---

## Core Responsibilities

`SatoHook` is responsible for:

* validating the intended ETH/SATO pool,
* rejecting unsupported pool configurations,
* preventing external liquidity additions,
* processing protocol buys,
* processing protocol sells,
* computing forward curve issuance,
* computing inverse curve redemption,
* tracking fair curve supply,
* tracking cumulative curve ETH,
* tracking protocol fees,
* enforcing cooldown rules,
* applying launch entropy adjustments,
* calling `SatoToken.mint()` during valid buys,
* calling `SatoToken.burn()` during valid sells.

The hook does **not** perform ERC-20 balance accounting directly.

Instead, it invokes the restricted mint and burn interface of `SatoToken` after protocol validation.

---

## Immutable Configuration

`SatoHook` stores immutable deployment configuration.

| Configuration   | Description                                                    |
| --------------- | -------------------------------------------------------------- |
| `POOL_MANAGER`  | Uniswap v4 PoolManager used for hook callbacks and settlement. |
| `SATO_TOKEN`    | SATO ERC-20 token contract.                                    |
| `ETH_CURRENCY`  | Native ETH currency wrapper.                                   |
| `SATO_CURRENCY` | SATO currency wrapper.                                         |
| `GENESIS_BLOCK` | Block number at hook deployment.                               |
| `GENESIS_HASH`  | Hash of the block immediately preceding hook deployment.       |

These values define the protocol environment and cannot be modified after deployment.

---

## Protocol Constants

The hook defines several locked constants.

| Constant                           | Purpose                                                   |
| ---------------------------------- | --------------------------------------------------------- |
| `K_SUPPLY`                         | Asymptotic supply cap of the bonding curve.               |
| `S`                                | Curve scale factor.                                       |
| `MAX_BUY_WEI`                      | Maximum ETH input allowed in a single buy.                |
| `COOLDOWN_BLOCKS`                  | Number of blocks between a wallet's buy and first sell.   |
| `POOL_FEE`                         | Required Uniswap v4 pool fee.                             |
| `ENTROPY_BLOCKS`                   | Number of blocks during which entropy adjustment applies. |
| `EXHAUSTION_THRESHOLD_NUMERATOR`   | Self-deprecation threshold numerator.                     |
| `EXHAUSTION_THRESHOLD_DENOMINATOR` | Self-deprecation threshold denominator.                   |
| `FEE_NUMERATOR`                    | Protocol fee numerator.                                   |
| `FEE_DENOMINATOR`                  | Protocol fee denominator.                                 |

Because these values are constants, they are not adjustable by governance or administrators.

---

## Protocol State

Protocol state evolves only through valid hook execution.

| Variable          | Purpose                                                     |
| ----------------- | ----------------------------------------------------------- |
| `ethCum`          | Cumulative fair-curve ETH after protocol fees.              |
| `totalMintedFair` | Canonical fair-curve circulating supply used by the curve.  |
| `selfDeprecated`  | Indicates whether new buys have been permanently disabled.  |
| `poolInitialized` | Indicates whether the configured pool has been initialized. |
| `lastBuyBlock`    | Records the latest buy block for each participant.          |
| `feesAccrued`     | Tracks cumulative ETH fees accrued by the hook.             |

These variables collectively represent the active monetary state of the protocol.

---

## Hook Permissions

`SatoHook` enables only the Uniswap v4 hook permissions required for protocol execution.

| Hook Callback           | Enabled | Purpose                                             |
| ----------------------- | ------- | --------------------------------------------------- |
| `beforeInitialize`      | Yes     | Validate the target pool before activation.         |
| `beforeAddLiquidity`    | Yes     | Reject external liquidity additions.                |
| `beforeSwap`            | Yes     | Execute protocol buy and sell logic.                |
| `beforeSwapReturnDelta` | Yes     | Return custom settlement deltas to the PoolManager. |

All other hook callbacks are intentionally disabled to reduce protocol complexity and attack surface.

During construction, the hook calls `Hooks.validateHookPermissions()` to verify that the deployed address matches the expected Uniswap v4 permission bitmap.

This protects against deploying the hook at an address with incorrect hook permission bits.

---

## Pool Initialization

`beforeInitialize()` validates the pool before protocol activation.

The pool must satisfy all required conditions:

* `currency0` must be native ETH.
* `currency1` must be SATO.
* the fee must equal `POOL_FEE`.
* the hook address must equal the deployed `SatoHook`.

If any condition fails, initialization reverts.

This prevents the hook from being used with an unintended pool configuration.

---

## Liquidity Model

SATO does not rely on externally supplied AMM liquidity.

`beforeAddLiquidity()` always reverts.

This means users cannot directly add liquidity to the ETH/SATO pool through the normal liquidity path.

Instead, the hook takes over swap execution and settles buys and sells through the deterministic bonding curve.

In practice:

* buys mint newly issued SATO through the curve,
* sells burn SATO through the inverse curve,
* ETH settlement occurs through the PoolManager,
* reserve accounting remains controlled by protocol logic.

External LP positions are not the source of protocol liquidity.

---

## PoolManager Execution Context

Most protocol execution occurs through Uniswap v4 PoolManager callbacks.

The hook restricts entrypoints using the PoolManager execution context.

This means buy and sell logic is not exposed as arbitrary public functions for direct user execution.

The PoolManager acts as the settlement coordinator, while the hook provides the monetary logic.

This separation allows the hook to use Uniswap v4 settlement mechanics without relying on normal AMM liquidity.

---

## Swapper Identity

`beforeSwap()` requires `hookData` to encode the actual swapper address.

The hook uses this address to:

* record the buyer's latest buy block,
* enforce cooldown rules,
* associate protocol execution with the initiating account.

If `hookData` is missing or too short, execution reverts.

The protocol intentionally avoids inferring the user identity only from intermediate routing contracts, because routers and PoolManager interactions may abstract away the actual user.

---

## Swap Entry Point

Every protocol trade enters through `beforeSwap()`.

The hook supports exact-input swaps only.

If an exact-output swap is attempted, execution reverts.

The swap direction determines the protocol path:

| Direction            | Meaning    | Protocol Path  |
| -------------------- | ---------- | -------------- |
| `zeroForOne = true`  | ETH → SATO | Buy execution  |
| `zeroForOne = false` | SATO → ETH | Sell execution |

Before execution continues, the hook validates:

* the pool configuration,
* the swap type,
* the swap direction,
* the presence of swapper identity,
* protocol state,
* path-specific constraints.

After validation, execution continues into either the buy path or the sell path.

# Buy Execution

A protocol buy occurs when native ETH is exchanged for newly issued SATO.

Unlike a conventional AMM swap, the protocol does not transfer existing tokens from a liquidity pool.

Instead, SatoHook calculates the amount of SATO to issue using the bonding curve and instructs the ERC-20 contract to mint the resulting tokens.

## Buy Execution Pipeline

```text
ETH Input
    │
    ▼
Validate Protocol State
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
Apply Entropy Adjustment
    │
    ▼
Mint SATO
    │
    ▼
Update Reserve State
    │
    ▼
Record Buy Block
    │
    ▼
Return Settlement Delta
```

A successful buy performs the following operations:

1. Verify that the protocol has not entered self-deprecation.
2. Reject buys exceeding the configured maximum size.
3. Calculate protocol fees.
4. Add the net ETH amount to the cumulative curve position.
5. Compute fair issuance using `Curve.mintFor()`.
6. Apply the entropy multiplier if the transaction occurs during the launch window.
7. Mint SATO through the ERC-20 contract.
8. Increase `ethCum`.
9. Increase `totalMintedFair`.
10. Record the buyer's latest block in `lastBuyBlock`.
11. Increase `feesAccrued`.
12. Return settlement deltas to the PoolManager.

No discretionary logic exists during this process.

Every transition is determined exclusively by immutable protocol rules.

---

# Sell Execution

A protocol sell occurs when SATO is redeemed for native ETH.

Unlike traditional AMMs, redemption is not performed against externally supplied liquidity.

Instead, ETH owed to the seller is calculated directly from the inverse bonding curve.

## Sell Execution Pipeline

```text
SATO Input
    │
    ▼
Validate Cooldown
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
Verify Reserve
    │
    ▼
Burn SATO
    │
    ▼
Transfer ETH
    │
    ▼
Update Curve State
```

A successful sell performs the following operations:

1. Verify the cooldown requirement.
2. Convert entropy-adjusted supply into canonical fair supply.
3. Calculate ETH redemption using `Curve.burnFor()`.
4. Calculate protocol fees.
5. Verify that sufficient ETH reserve exists.
6. Burn SATO through the ERC-20 contract.
7. Settle ETH through the PoolManager.
8. Decrease `ethCum`.
9. Decrease `totalMintedFair`.
10. Increase `feesAccrued`.
11. Return settlement deltas.

Protocol sells never depend on external liquidity providers.

Every redemption is calculated directly from the deterministic inverse curve.

---

# Fair Supply Model

The protocol intentionally distinguishes between actual token supply and canonical curve supply.

| Value                  | Description                                               |
| ---------------------- | --------------------------------------------------------- |
| ERC-20 `totalSupply()` | Actual number of SATO tokens currently in circulation.    |
| `totalMintedFair`      | Canonical supply used for all bonding curve calculations. |

These values may temporarily differ because the launch entropy mechanism can increase or decrease the amount of SATO received during early buys.

The bonding curve, however, always operates on the canonical fair supply.

This guarantees deterministic pricing throughout the lifetime of the protocol.

---

# Entropy Mechanism

During the initial launch phase the protocol applies a bounded entropy adjustment.

Entropy introduces a small deterministic variation in token issuance without modifying the underlying bonding curve.

Inputs include:

* previous block hash,
* swapper address,
* ETH input amount.

The resulting multiplier remains bounded within approximately:

```text
0.90×
   to
1.10×
```

The entropy window exists only during the configured launch period.

Once the entropy window expires, protocol issuance exactly matches the bonding curve.

Entropy therefore affects only early issuance and never changes long-term monetary policy.

---

# Cooldown Mechanism

The hook records the latest successful buy block for every participant.

This value is stored in:

```text
lastBuyBlock
```

Before processing a sell, the hook verifies that the configured cooldown has elapsed.

The current implementation requires a one-block delay.

Cooldown enforcement reduces immediate buy-then-sell behavior while remaining completely deterministic.

No administrator participates in cooldown enforcement.

---

# Reserve Accounting

Unlike conventional liquidity pools, reserve accounting is maintained internally by the protocol.

Each successful buy increases the reserve represented by the bonding curve.

Each successful sell reduces that reserve according to the inverse curve.

The hook exposes reserve backing through:

```text
curveReserveEth()
```

Conceptually, the reserve is represented as:

```text
Protocol ETH Balance
        −
 Accrued Protocol Fees
```

Reserve accounting is therefore independent from protocol fee accumulation.

External liquidity providers do not contribute to reserve backing.

The reserve exists solely as a consequence of deterministic protocol execution.

# Fee Accounting

Protocol fees are collected during both buy and sell execution.

The fee ratio is fixed by immutable protocol constants and is applied automatically during settlement.

Accumulated fees are tracked through:

```text
feesAccrued
```

Fees are intentionally accounted for separately from reserve backing.

This distinction ensures that protocol revenue is never interpreted as collateral supporting token redemption.

---

# Self-Deprecation

The protocol includes an automatic self-deprecation mechanism.

When the canonical fair supply reaches the configured exhaustion threshold, the hook permanently disables additional buys.

After self-deprecation:

* New buy operations revert.
* Existing holders may continue selling.
* Reserve redemption remains available.
* No administrator intervention is required.

Self-deprecation affects only forward issuance.

It does not prevent existing holders from exiting through the inverse bonding curve.

Once activated, self-deprecation is irreversible.

---

# ETH Flow

ETH enters the protocol through successful buy execution.

```text
Buyer
   │
   ▼
PoolManager
   │
   ▼
SatoHook Reserve
```

ETH leaves the protocol through successful sell execution.

```text
SatoHook Reserve
        │
        ▼
PoolManager
        │
        ▼
Seller
```

The hook does not expose arbitrary withdrawal mechanisms for protocol reserves.

Reserve movements occur only through deterministic protocol execution.

---

# Access Control

SatoHook intentionally minimizes privileged execution.

Rather than using owner-controlled administration, execution authority is derived from protocol context.

| Operation           | Access Restriction            |
| ------------------- | ----------------------------- |
| Pool initialization | Valid pool configuration only |
| Buy execution       | PoolManager callback          |
| Sell execution      | PoolManager callback          |
| Token minting       | Locked protocol minter        |
| Token burning       | Locked protocol minter        |

The hook validates execution context before any protocol state transition.

---

# Security Model

SatoHook is designed to minimize trusted assumptions.

Important security properties include:

* Immutable deployment configuration.
* One-time pool initialization.
* Validated hook permissions.
* PoolManager-only execution.
* Deterministic bonding curve execution.
* External liquidity rejection.
* Reserve-backed redemption.
* Deterministic cooldown enforcement.
* Bounded entropy adjustment.
* Locked ERC-20 mint authorization.
* Locked ERC-20 burn authorization.
* No discretionary monetary policy.

The hook is not an administrative controller.

It is a deterministic execution engine that applies immutable protocol rules.

---

# Relationship with SatoToken

SatoHook and SatoToken intentionally separate monetary execution from token accounting.

| Component   | Responsibility            |
| ----------- | ------------------------- |
| `SatoHook`  | Monetary execution        |
| `SatoToken` | ERC-20 accounting         |
| `Curve`     | Bonding curve mathematics |

The hook never modifies balances directly.

Instead, after validating protocol rules, it invokes the restricted `mint()` and `burn()` interface exposed by the ERC-20 contract.

This separation improves modularity, simplifies auditing, and isolates accounting from monetary policy.

---

# Scanner Interpretation

Automated security scanners may detect privileged mint and burn functionality.

Viewed in isolation, these functions can resemble owner-controlled token issuance.

Within the complete protocol architecture, however, minting and burning are deterministic protocol operations.

Important architectural properties include:

* The ERC-20 minter is permanently locked.
* Minting occurs only during valid protocol buys.
* Burning occurs only during valid protocol sells.
* Redemption follows the deterministic inverse bonding curve.
* Monetary policy is enforced by immutable protocol logic.

Scanner findings should therefore be interpreted together with the complete protocol architecture rather than isolated function signatures.

Additional discussion is available in:

```text
docs/security/05_False_Positive_Analysis.md
```

---

# Design Principles

The hook is intentionally designed around the following principles:

* Deterministic execution.
* Immutable monetary rules.
* Minimal trusted assumptions.
* Transparent reserve accounting.
* Modular protocol architecture.
* Separation of accounting and monetary policy.

These principles guide every protocol state transition.

---

# Summary

SatoHook is the deterministic execution layer of the SATO protocol.

It validates protocol rules, executes the bonding curve, maintains reserve accounting, tracks canonical monetary state, enforces cooldown and entropy rules, and coordinates ERC-20 mint and burn operations.

Rather than relying on privileged administrators or discretionary governance, protocol behavior is enforced entirely through immutable smart contract logic.

Together with the Curve mathematical library and the SatoToken accounting layer, SatoHook forms the deterministic execution layer of the SATO monetary protocol.

---

## Related Documentation

* `03_ERC20.md`
* `05_Bonding_Curve.md`
* `06_Reserve_Model.md`
* `docs/security/01_Overview.md`
* `docs/security/05_False_Positive_Analysis.md`

---

**END OF FILE**
