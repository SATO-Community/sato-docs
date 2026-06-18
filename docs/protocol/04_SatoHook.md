# SatoHook Protocol Contract

## Overview

`SatoHook` is the protocol execution layer of SATO.

While the SATO ERC-20 contract handles token accounting, `SatoHook` handles the monetary engine of the protocol.

It is responsible for:

* Buy execution
* Sell execution
* Bonding curve interaction
* ETH reserve accounting
* Protocol fee accounting
* Mint authorization
* Burn authorization
* Uniswap v4 hook integration

In the SATO architecture, the ERC-20 token is the accounting layer, while `SatoHook` is the protocol layer.

---

## Core Design

`SatoHook` uses Uniswap v4 hook mechanics to take over swap execution.

The contract does not rely on traditional AMM liquidity.

Instead, liquidity additions are forbidden, and the hook acts as the counterparty for buys and sells through deterministic bonding curve logic.

This means:

* ETH to SATO swaps are processed as protocol buys.
* SATO to ETH swaps are processed as protocol sells.
* SATO is minted during valid buys.
* SATO is burned during valid sells.
* ETH reserve accounting is handled by the hook.

---

## Architecture

```text
User
  │
  ▼
Router
  │
  ▼
Uniswap v4 PoolManager
  │
  ▼
SatoHook
  │
  ├── Curve.mintFor()
  ├── Curve.burnFor()
  ├── SatoToken.mint()
  └── SatoToken.burn()
```

The PoolManager provides the settlement layer.

`SatoHook` provides the monetary logic.

---

## Locked Parameters

The contract defines several locked protocol parameters.

| Parameter         | Purpose                                                    |
| ----------------- | ---------------------------------------------------------- |
| `K_SUPPLY`        | Asymptotic supply cap of the curve.                        |
| `S`               | Curve scale factor.                                        |
| `MAX_BUY_WEI`     | Maximum ETH input on a single buy.                         |
| `COOLDOWN_BLOCKS` | Minimum block delay between a wallet's buy and first sell. |
| `POOL_FEE`        | Configured Uniswap v4 pool fee.                            |
| `ENTROPY_BLOCKS`  | Initial block window for entropy multiplier.               |
| `FEE_NUMERATOR`   | Protocol fee numerator.                                    |
| `FEE_DENOMINATOR` | Protocol fee denominator.                                  |

These parameters are constants and cannot be modified after deployment.

---

## Immutable References

`SatoHook` stores immutable references to:

* Uniswap v4 `POOL_MANAGER`
* SATO ERC-20 token
* Native ETH currency
* SATO currency wrapper
* Deployment block
* Previous block hash

These immutable values define the protocol environment at deployment.

---

## Storage Variables

| Variable          | Purpose                                                     |
| ----------------- | ----------------------------------------------------------- |
| `ethCum`          | Cumulative fair-curve ETH after fees.                       |
| `totalMintedFair` | Canonical fair-curve circulating supply.                    |
| `selfDeprecated`  | Indicates whether the curve has entered no-more-buys mode.  |
| `poolInitialized` | Indicates whether the configured pool has been initialized. |
| `lastBuyBlock`    | Tracks each address's most recent buy block.                |
| `feesAccrued`     | Tracks cumulative ETH fees accumulated by the hook.         |

These variables define the active protocol state.

---

## Hook Permissions

`SatoHook` declares the following active Uniswap v4 permissions:

* `beforeInitialize`
* `beforeAddLiquidity`
* `beforeSwap`
* `beforeSwapReturnDelta`

All other hook permissions are disabled.

The constructor validates that the deployed hook address contains the correct Uniswap v4 permission bits.

---

## Pool Initialization

`beforeInitialize()` validates that the initialized pool matches the expected SATO market.

The pool must satisfy:

* `currency0` is native ETH
* `currency1` is SATO
* fee equals `POOL_FEE`
* hook address equals `SatoHook`

If any condition fails, the function reverts.

This prevents the hook from being attached to an unintended pool configuration.

---

## Liquidity Additions

`beforeAddLiquidity()` always reverts.

This means external liquidity cannot be added directly to the pool.

The protocol intentionally avoids traditional AMM liquidity and instead routes swap behavior through hook-controlled bonding curve execution.

---

## Swap Execution

`beforeSwap()` is the core trading entrypoint.

It supports only exact-input swaps.

If an exact-output swap is attempted, the function reverts.

The swap direction determines protocol behavior:

* `zeroForOne = true` means ETH to SATO buy.
* `zeroForOne = false` means SATO to ETH sell.

The hook requires `hookData` to include the swapper address.

This address is used for cooldown tracking.

---

## Buy Flow

A buy occurs when ETH is swapped for SATO.

Execution flow:

1. Validate buy amount.
2. Reject buys after self-deprecation.
3. Compute protocol fee.
4. Send net ETH into the curve calculation.
5. Compute fair SATO output with `Curve.mintFor()`.
6. Apply entropy multiplier if inside the entropy window.
7. Mint SATO to the PoolManager.
8. Take ETH from the PoolManager into the hook.
9. Update `ethCum`.
10. Update `totalMintedFair`.
11. Accrue fee revenue.
12. Record `lastBuyBlock`.
13. Check self-deprecation threshold.
14. Return `BeforeSwapDelta`.

The ERC-20 token supply increases only after the hook executes a valid protocol buy.

---

## Sell Flow

A sell occurs when SATO is swapped for ETH.

Execution flow:

1. Enforce cooldown against the seller's last buy block.
2. Convert actual SATO input into fair-curve units.
3. Compute ETH owed using `Curve.burnFor()`.
4. Compute fee.
5. Verify ETH reserves are sufficient.
6. Take SATO from the PoolManager.
7. Burn SATO through the ERC-20 contract.
8. Settle ETH to the PoolManager.
9. Reduce `ethCum`.
10. Reduce `totalMintedFair`.
11. Accrue fee revenue.
12. Return `BeforeSwapDelta`.

The ERC-20 token supply decreases only after the hook executes a valid protocol sell.

---

## Fair Supply vs Actual Supply

`totalMintedFair` is separate from the ERC-20 `totalSupply()`.

This distinction exists because early buys may receive an entropy multiplier.

The fair supply represents the canonical curve position.

The actual token supply may differ due to entropy-adjusted minting.

During sells, the hook converts actual SATO input into fair-curve units before applying the inverse curve.

This keeps redemption pricing tied to the canonical curve position.

---

## Entropy Window

During the first `ENTROPY_BLOCKS`, buys receive an entropy multiplier.

The multiplier is derived from:

* previous block hash
* swapper address
* ETH input amount

The multiplier range is approximately:

```text
0.90x to 1.10x
```

After the entropy window ends, buys receive the fair curve amount without multiplier adjustment.

---

## Cooldown

The hook records each swapper's last buy block.

A seller cannot sell within the configured cooldown window after buying.

This mechanism is implemented through `lastBuyBlock`.

The configured cooldown is one block.

---

## Fee Accounting

The hook skims a fee on both buys and sells.

The fee is calculated using:

```text
FEE_NUMERATOR / FEE_DENOMINATOR
```

In the current implementation this corresponds to 0.3%.

Fees accumulate in `feesAccrued`.

The curve reserve view subtracts accumulated fees from the contract ETH balance.

---

## Curve Reserve

`curveReserveEth()` returns:

```text
address(this).balance - feesAccrued
```

This value represents ETH backing the inverse curve for sells.

Accumulated fees are excluded from the displayed curve reserve.

---

## Self-Deprecation

The hook includes a self-deprecation mechanism.

When `totalMintedFair` reaches the configured exhaustion threshold, `selfDeprecated` becomes true.

After self-deprecation:

* buys are rejected
* sells may continue

This mechanism limits further forward-curve issuance after the threshold is reached.

---

## ETH Handling

The contract can receive ETH.

ETH enters the hook primarily through PoolManager settlement during buy execution.

ETH exits the hook through PoolManager settlement during sell execution.

The contract comments explicitly discourage direct ETH transfers.

---

## Error Conditions

The hook defines custom errors for invalid protocol conditions.

Examples include:

* Non-PoolManager caller
* Invalid pool configuration
* Attempted liquidity addition
* Oversized buy
* Cooldown violation
* Buy after self-deprecation
* Exact-output swap attempt
* Missing swapper identity
* Insufficient ETH reserves

These errors make failure conditions explicit and easier to analyze.

---

## Security Notes

`SatoHook` is not a discretionary owner.

It is the deterministic protocol execution layer.

The hook can mint and burn SATO only through the ERC-20 contract because it is the locked minter.

This mint/burn capability is required for protocol operation and should be evaluated in the context of the full hook design.

Important security properties include:

* Hook entrypoints are restricted to the PoolManager.
* Pool initialization is validated.
* Direct liquidity additions are forbidden.
* Exact-output swaps are unsupported.
* Buys and sells follow deterministic curve logic.
* Minting and burning occur only as part of protocol execution.

---

## Relationship with ERC-20

The ERC-20 contract does not implement monetary policy.

`SatoHook` determines when minting and burning are valid.

The ERC-20 contract records the resulting token accounting changes.

This separation is central to the protocol architecture.

---

## Summary

`SatoHook` is the monetary engine of the SATO protocol.

It replaces traditional AMM liquidity with deterministic bonding curve execution through Uniswap v4 hook mechanics.

The hook manages buy and sell flows, ETH reserve accounting, fee accounting, entropy adjustment, cooldown enforcement, and self-deprecation logic.

Together with the SATO ERC-20 accounting layer, it forms a protocol architecture designed around deterministic execution, transparent reserve accounting, and minimized administrative discretion.
