# SatoHook Protocol Contract

## Overview

`SatoHook` is the monetary execution layer of the SATO protocol.

The SATO ERC-20 contract records balances, transfers, allowances, and total supply. `SatoHook` defines when SATO can be minted, when SATO can be burned, how ETH reserves are updated, and how curve-routed buys and sells are settled.

In the deployed architecture:

| Component | Role |
| --- | --- |
| `SatoToken` | ERC-20 accounting |
| `SatoHook` | Protocol execution |
| `Curve` | Bonding curve mathematics |
| Uniswap v4 `PoolManager` | Settlement layer |

## Deployed Contract

| Property | Value |
| --- | --- |
| Network | Ethereum Mainnet |
| Contract | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` |
| Contract name | `SatoHook` |
| Compiler | Solidity `0.8.26` |
| Verification | Etherscan source code verified, exact match |
| SATO token | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| Uniswap v4 PoolManager | `0x000000000004444c5dc75cB358380D2e3dE08A90` |

## Design Purpose

`SatoHook` is designed to enforce deterministic monetary behavior without discretionary administration.

The hook does not manage a treasury, upgrade monetary rules, pause the protocol, whitelist users, blacklist users, or rely on external liquidity providers for the curve pool.

Instead, it validates swaps, applies the bonding curve, coordinates mint and burn operations through `SatoToken`, and maintains ETH reserve accounting.

## System Architecture

```text
                 User / Router
                      |
                      v
              Uniswap v4 PoolManager
                      |
                      v
                  SatoHook
             +--------+--------+
             |                 |
             v                 v
          Curve             SatoToken
     pricing math       ERC-20 accounting
             |                 |
             +--------+--------+
                      |
                      v
        Mint / Burn and ETH Reserve Updates
```

The hook uses Uniswap v4 hook callbacks as settlement rails. It uses `BeforeSwapDelta` to take over swap accounting for the curve pool.

## Core Responsibilities

`SatoHook` is responsible for:

- validating the configured ETH/SATO pool
- rejecting unsupported pool configurations
- preventing external liquidity additions
- processing curve-based buys
- processing curve-based sells
- computing forward curve issuance
- computing inverse curve redemption
- tracking cumulative curve ETH
- tracking fair curve supply
- tracking protocol fees
- enforcing buy/sell cooldown rules
- applying launch entropy adjustments
- calling `SatoToken.mint()` during valid buys
- calling `SatoToken.burn()` during valid sells
- holding ETH reserves used for curve redemptions

`SatoHook` does not perform ERC-20 balance accounting directly. Token accounting remains inside `SatoToken`.

## Immutable Configuration

| Configuration | Description |
| --- | --- |
| `POOL_MANAGER` | Uniswap v4 PoolManager used for hook callbacks and settlement. |
| `SATO_TOKEN` | SATO ERC-20 token contract. |
| `ETH_CURRENCY` | Native ETH currency wrapper. |
| `SATO_CURRENCY` | SATO currency wrapper. |
| `GENESIS_BLOCK` | Hook deployment block number. |
| `GENESIS_HASH` | Hash of the block immediately preceding hook deployment. |

These values are fixed at deployment and cannot be modified afterward.

## Protocol Constants

| Constant | Value | Purpose |
| --- | --- | --- |
| `K_SUPPLY` | `21,000,000e18` | Asymptotic fair supply of the bonding curve. |
| `S` | `500 ether` | Curve scale parameter. |
| `MAX_BUY_WEI` | `5 ether` | Maximum ETH input for a single buy. |
| `COOLDOWN_BLOCKS` | `1` | Blocks required between a wallet's buy and first sell. |
| `POOL_FEE` | `3000` | Uniswap v4 fee value, equal to `0.3%`. |
| `ENTROPY_BLOCKS` | `100` | Initial launch window for entropy multiplier. |
| `EXHAUSTION_THRESHOLD_NUMERATOR` | `99` | Self-deprecation threshold numerator. |
| `EXHAUSTION_THRESHOLD_DENOMINATOR` | `100` | Self-deprecation threshold denominator. |
| `FEE_NUMERATOR` | `30` | Protocol fee numerator. |
| `FEE_DENOMINATOR` | `10000` | Protocol fee denominator. |

The protocol fee is `0.3%`.

## Protocol State

| State variable | Purpose |
| --- | --- |
| `ethCum` | Cumulative ETH committed to the fair curve after fees. |
| `totalMintedFair` | Canonical fair-curve supply used for curve accounting. |
| `selfDeprecated` | Indicates whether new buys are disabled after curve exhaustion threshold. |
| `poolInitialized` | Records whether the configured pool has been initialized. |
| `lastBuyBlock` | Tracks each wallet's last curve buy block for cooldown enforcement. |
| `feesAccrued` | Tracks accumulated protocol fees. |

## Hook Permissions

The deployed hook declares only the permissions required for protocol operation.

| Hook permission | Status |
| --- | --- |
| `beforeInitialize` | enabled |
| `beforeAddLiquidity` | enabled |
| `beforeSwap` | enabled |
| `beforeSwapReturnDelta` | enabled |
| all other hook permissions | disabled |

`beforeAddLiquidity` is enabled so the hook can reject liquidity additions. The function always reverts with `LiquidityAdditionsForbidden()`.

## Pool Initialization

The hook accepts only the configured ETH/SATO curve pool.

During `beforeInitialize`, the hook validates:

- `currency0` is native ETH
- `currency1` is SATO
- pool fee is `3000`
- hook address is the deployed `SatoHook`

If any condition fails, initialization reverts with `InvalidPool()`.

## Liquidity Model

The curve pool does not use ordinary external AMM liquidity.

External liquidity additions are permanently forbidden.

Instead, `SatoHook` acts as the counterparty for curve-routed swaps:

- buys mint SATO through the forward curve
- sells burn SATO through the inverse curve
- ETH received from buys is held by the hook
- ETH paid to sellers comes from the hook reserve

Secondary AMM pools may exist separately, but trades through secondary pools do not mint or burn SATO and do not update the curve reserve.

## Swap Entry Point

The core trading entry point is `beforeSwap`.

The hook supports exact-input swaps only. Exact-output swaps revert with `ExactOutputUnsupported()`.

The hook requires `hookData` to include the swapper address. This address is used for cooldown tracking.

For the ETH/SATO curve pool:

- `zeroForOne == true` executes an ETH -> SATO buy
- `zeroForOne == false` executes a SATO -> ETH sell

## Buy Execution

A buy is an exact-input ETH -> SATO swap through the curve pool.

```text
User provides ETH
      |
      v
Router pre-settles ETH to PoolManager
      |
      v
SatoHook receives beforeSwap
      |
      v
Protocol fee is separated
      |
      v
Curve calculates fair SATO output
      |
      v
Launch entropy may adjust actual minted amount
      |
      v
SatoHook mints SATO to PoolManager
      |
      v
PoolManager routes SATO to buyer
      |
      v
SatoHook takes ETH into reserve
      |
      v
Curve state and fee state update
```

Buy validation:

- `ethIn` must not exceed `5 ETH`
- `selfDeprecated` must be false
- swap must be exact-input
- pool must be the configured ETH/SATO pool

Buy effects:

- fee is calculated from ETH input
- post-fee ETH advances the fair curve
- fair SATO output is calculated by `Curve.mintFor()`
- actual minted amount may include launch entropy adjustment
- SATO is minted to the PoolManager for settlement
- ETH is taken into the hook contract
- `ethCum` increases by post-fee ETH
- `totalMintedFair` increases by fair SATO output
- `feesAccrued` increases by the fee
- `lastBuyBlock[swapper]` is updated

## Sell Execution

A sell is an exact-input SATO -> ETH swap through the curve pool.

```text
User provides SATO
      |
      v
Router pre-settles SATO to PoolManager
      |
      v
SatoHook receives beforeSwap
      |
      v
Actual SATO input is converted to fair-curve units
      |
      v
Inverse curve calculates ETH redemption
      |
      v
Protocol fee is separated
      |
      v
SatoHook takes and burns SATO
      |
      v
SatoHook settles ETH to PoolManager
      |
      v
PoolManager routes ETH to seller
      |
      v
Curve state and fee state update
```

Sell validation:

- wallet must not be within the cooldown window after its last buy
- hook ETH balance must be sufficient for the redemption
- swap must be exact-input
- pool must be the configured ETH/SATO pool

Sell effects:

- sold SATO is converted into fair-curve units
- ETH redemption is calculated by `Curve.burnFor()`
- fee is deducted from raw ETH redemption
- SATO is taken from the PoolManager and burned
- ETH is settled to the PoolManager for the seller
- `ethCum` is reduced by the raw curve redemption
- `totalMintedFair` is reduced by the fair SATO input
- `feesAccrued` increases by the fee

## Fair Supply Model

The hook tracks `totalMintedFair` separately from actual ERC-20 total supply.

This is necessary because the launch entropy mechanism may cause actual minted supply to differ from the fair curve output during the first `100` blocks.

For sell execution, actual SATO input is converted into fair-curve units:

```text
satoFairIn = satoIn * totalMintedFair / actualSupply
```

This keeps inverse curve redemption tied to the canonical fair curve position rather than raw token supply.

## Entropy Mechanism

During the first `100` blocks after hook deployment, buys receive an entropy multiplier.

The multiplier is derived from:

- previous block hash
- swapper address
- ETH input amount

The multiplier range is:

```text
0.9x to 1.1x
```

After the entropy window ends, buys mint exactly the fair curve output.

The whitepaper notes that this launch window has closed. From that point onward, the curve operates deterministically.

## Cooldown Mechanism

The hook enforces a cooldown between a wallet's buy and first sell through the curve.

If a wallet attempts to sell in the same block as its last curve buy, the sell reverts with `CooldownActive()`.

This rule is intended to reduce same-block buy/sell behavior and flash-loan-style curve abuse.

## Reserve Accounting

ETH received from buys is held by the hook contract.

The sell-side curve reserve is exposed as:

```text
curveReserveEth = address(this).balance - feesAccrued
```

The reserve backs inverse-curve redemptions.

ETH leaves the hook through curve sell execution when SATO is burned and ETH is settled back through the PoolManager.

The hook has a payable `receive()` function so it can receive ETH from the PoolManager and settled buyer funds. Users should not treat direct ETH transfers to the hook as protocol buys.

## Fee Accounting

A `0.3%` protocol fee is applied on both buys and sells.

On buys:

```text
fee = ethIn * 30 / 10000
ethToCurve = ethIn - fee
```

On sells:

```text
fee = ethRaw * 30 / 10000
ethOut = ethRaw - fee
```

Fees are tracked in `feesAccrued`.

The whitepaper describes these fees as accumulating inside the hook permanently rather than functioning as a treasury withdrawal mechanism.

## Self-Deprecation

The hook disables new buys when the fair curve reaches the exhaustion threshold.

The threshold is:

```text
totalMintedFair >= 99% of K_SUPPLY
```

When this threshold is reached, `selfDeprecated` becomes true.

After self-deprecation:

- new buys revert with `SelfDeprecatedNoBuys()`
- sells continue to be supported against available curve reserves

This allows the protocol to transition from active issuance toward dormancy while preserving redemption behavior.

## Access Control

The hook does not expose owner-based administrative controls.

Hook entry points are protected by the Uniswap v4 PoolManager execution context. Functions intended for pool callbacks require `msg.sender` to be the configured `POOL_MANAGER`.

The hook does not implement:

- owner withdrawals
- governance upgrades
- pause controls
- blacklist controls
- whitelist controls
- external liquidity management
- discretionary minting

Minting and burning occur only through the hook's interaction with the locked `SatoToken` minter path.

## Relationship with SatoToken

`SatoHook` and `SatoToken` intentionally separate monetary execution from token accounting.

| Component | Responsibility |
| --- | --- |
| `SatoHook` | Monetary execution |
| `SatoToken` | ERC-20 accounting |
| `Curve` | Bonding curve mathematics |

During buys, `SatoHook` calls `SatoToken.mint()`.

During sells, `SatoHook` calls `SatoToken.burn()`.

The ERC-20 contract does not decide when minting or burning should occur. The hook executes protocol rules and then uses the token contract to record the resulting supply change.

## Scanner Interpretation

Automated security scanners may identify mint and burn functions in the ERC-20 contract.

Viewed in isolation, those functions can resemble privileged balance-changing functionality.

In the full SATO architecture:

- the ERC-20 minter is permanently locked
- the locked minter is the deployed `SatoHook`
- minting occurs during valid curve buys
- burning occurs during valid curve sells
- redemption follows deterministic inverse curve math
- monetary behavior is enforced by verified contract logic

Scanner findings should therefore be interpreted together with the deployed hook architecture rather than only the ERC-20 function names.

## Security Notes

This document describes the verified protocol architecture.

It is not an audit.

Important review areas include:

- Uniswap v4 hook execution behavior
- PoolManager settlement assumptions
- router pre-settlement behavior
- curve math precision
- reserve accounting
- sell-side redemption behavior
- scanner interpretation of mint and burn functions

## Summary

`SatoHook` is the deterministic monetary execution layer of the SATO protocol.

It validates the configured curve pool, rejects external liquidity additions, executes curve buys and sells, maintains fair curve state, tracks fees, holds ETH reserves, and coordinates protocol-authorized mint and burn operations through `SatoToken`.

Together with `SatoToken` and `Curve`, `SatoHook` enforces SATO issuance and redemption through immutable smart contract logic rather than discretionary administration.

## Related Documentation

- `docs/protocol/01_Protocol_Overview.md`
- `docs/protocol/02_Architecture.md`
- `docs/protocol/03_ERC20.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/security/05_False_Positive_Analysis.md`
