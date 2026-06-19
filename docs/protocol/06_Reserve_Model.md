# Reserve Model

## Overview

SATO uses an on-chain reserve model.

ETH committed through curve buys is held by the `SatoHook` contract. SATO sold through the curve is burned, and ETH is redeemed from the hook's reserve according to the inverse bonding curve.

The reserve is not an off-chain asset, treasury account, multisig balance, or discretionary pool. It is ETH held by the deployed hook contract.

## Core Principle

The pool is the reserve.

When SATO is minted through the curve, ETH enters the hook contract. When SATO is burned through the curve, ETH leaves the hook contract through protocol execution.

The reserve exists on-chain and can be independently inspected.

## Reserve Contract

| Component | Address |
| --- | --- |
| SatoHook | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` |
| SATO token | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| Uniswap v4 PoolManager | `0x000000000004444c5dc75cB358380D2e3dE08A90` |

`SatoHook` holds ETH used for curve redemptions.

## Reserve Definition

The hook exposes the sell-side reserve as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

In contract form:

```solidity
function curveReserveEth() external view returns (uint256) {
    return address(this).balance - feesAccrued;
}
```

This value represents the ETH available to back inverse-curve redemptions, excluding accumulated protocol fees.

## Buy-Side Reserve Flow

A curve buy is an exact-input ETH -> SATO swap through the hook pool.

```text
Buyer provides ETH
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
Post-fee ETH advances the bonding curve
      |
      v
SATO is minted to PoolManager
      |
      v
SATO is routed to buyer
      |
      v
ETH is taken into SatoHook
      |
      v
Reserve and fee accounting update
```

On a buy:

```text
fee = ethIn * 30 / 10000
ethToCurve = ethIn - fee
```

State updates include:

```text
ethCum += ethToCurve
totalMintedFair += fairSato
feesAccrued += fee
```

The hook receives the full ETH input. The post-fee portion advances the curve, while the fee portion is tracked as accrued fees.

## Sell-Side Reserve Flow

A curve sell is an exact-input SATO -> ETH swap through the hook pool.

```text
Seller provides SATO
      |
      v
Router pre-settles SATO to PoolManager
      |
      v
SatoHook receives beforeSwap
      |
      v
SATO input is converted to fair-curve units
      |
      v
Inverse curve calculates raw ETH redemption
      |
      v
Protocol fee is separated
      |
      v
SATO is taken and burned
      |
      v
ETH is settled to PoolManager
      |
      v
ETH is routed to seller
      |
      v
Reserve and curve state update
```

On a sell:

```text
ethRaw = Curve.burnFor(totalMintedFair, satoFairIn)
fee = ethRaw * 30 / 10000
ethOut = ethRaw - fee
```

State updates include:

```text
ethCum -= ethRaw
totalMintedFair -= satoFairIn
feesAccrued += fee
```

The seller receives `ethOut`.

## Fee Treatment

A `0.3%` protocol fee is applied on both buys and sells.

Fees are tracked in:

```text
feesAccrued
```

The whitepaper describes these fees as accumulating inside the hook permanently rather than functioning as a withdrawable treasury.

The reserve view excludes fees:

```text
curveReserveEth = hook ETH balance - feesAccrued
```

This separates curve redemption backing from accumulated fee accounting.

## ETH Custody

ETH custody is held by the deployed `SatoHook` contract.

The hook does not expose an owner withdrawal function.

The hook does not expose a governance withdrawal function.

The hook does not expose a treasury transfer function.

ETH leaves the hook during sell execution when the hook settles ETH through the Uniswap v4 `PoolManager`.

## Redemption Backing

Curve redemptions are backed by the hook's ETH balance.

Before settling ETH to a seller, the hook checks that it has sufficient ETH for the redemption:

```text
address(SatoHook).balance >= ethOut
```

If insufficient ETH is available, the sell reverts with:

```text
InsufficientEthReserves()
```

## Fair Supply and Reserve Accounting

The reserve model uses the same fair-supply accounting as the bonding curve.

`totalMintedFair` is the canonical fair-curve supply.

Actual ERC-20 total supply may differ from fair supply because of the launch entropy mechanism. During sell execution, actual SATO input is converted into fair-curve units:

```text
satoFairIn = satoIn * totalMintedFair / actualSupply
```

This ensures redemption calculations are based on the canonical curve state.

## ETH Cumulative State

The hook tracks:

```text
ethCum
```

`ethCum` is cumulative ETH committed to the fair curve after buy-side fees and after sell-side curve retractions.

On buys, `ethCum` increases by post-fee ETH.

On sells, `ethCum` decreases by raw inverse-curve redemption.

If the raw redemption exceeds `ethCum`, the hook saturates `ethCum` to zero.

## Direct ETH Transfers

The hook includes a payable `receive()` function so it can receive ETH as part of PoolManager settlement and buyer flow.

Direct ETH transfers to the hook should not be treated as curve buys.

A protocol buy requires swap execution through the configured curve pool.

Only hook-routed swaps update curve state, mint SATO, burn SATO, or execute redemption logic.

## Secondary Markets

Secondary markets are separate from the reserve model.

A trade through a secondary AMM pool:

- does not mint SATO
- does not burn SATO
- does not move ETH into the curve reserve
- does not redeem ETH from the curve reserve
- does not update `ethCum`
- does not update `totalMintedFair`

The reserve model applies only to swaps routed through the ETH/SATO curve pool handled by `SatoHook`.

## Reserve Transparency

The reserve can be reviewed on-chain through:

- the ETH balance of `SatoHook`
- `feesAccrued`
- `curveReserveEth()`
- `ethCum`
- `totalMintedFair`
- SATO ERC-20 `totalSupply()`
- mint and burn events
- PoolManager settlement transactions

This allows independent verification of reserve state and curve execution.

## Design Summary

The SATO reserve model is designed around transparent on-chain custody.

ETH enters through curve buys.

ETH exits through curve sells.

SATO is minted only through curve issuance.

SATO is burned only through curve redemption.

The reserve is held by contract logic rather than an administrator, treasury, or off-chain custodian.

## Security Notes

This document describes the verified reserve architecture.

It is not an audit.

Important review areas include:

- hook ETH balance accounting
- fee accounting
- PoolManager settlement behavior
- router pre-settlement assumptions
- inverse curve redemption math
- direct ETH transfer edge cases
- scanner interpretation of mint and burn behavior

## Related Documentation

- `docs/protocol/01_Protocol_Overview.md`
- `docs/protocol/02_Architecture.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/security/03_Trust_Model.md`
- `docs/security/05_False_Positive_Analysis.md`
