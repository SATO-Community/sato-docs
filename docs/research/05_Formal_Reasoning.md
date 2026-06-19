# Formal Reasoning

## Overview

This document outlines the logical properties, state transitions, and reasoning framework used to describe the SATO protocol.

It is not a formal verification report, mathematical proof, or audit.

Instead, it documents how the verified contracts are intended to preserve deterministic monetary behavior through explicit state variables, restricted authorization paths, and on-chain reserve accounting.

## Reasoning Scope

The reasoning scope covers the core protocol components:

| Component | Role |
| --- | --- |
| `SatoToken` | ERC-20 accounting and locked mint/burn authorization |
| `SatoHook` | Monetary execution, curve settlement, reserve custody |
| `Curve` | Deterministic forward and inverse curve math |
| `SatoSwapRouter` | Direct curve routing helper |
| Uniswap v4 `PoolManager` | Settlement layer |

The router is included only as an execution path. It does not define monetary policy.

## Core State Variables

The protocol's monetary reasoning depends primarily on the following state:

| Variable | Contract | Meaning |
| --- | --- | --- |
| `minter` | `SatoToken` | Address authorized to call `mint()` and `burn()`. |
| `ethCum` | `SatoHook` | Cumulative ETH committed to the fair curve after fees and sell-side retractions. |
| `totalMintedFair` | `SatoHook` | Canonical fair-curve supply used for curve accounting. |
| `feesAccrued` | `SatoHook` | Accumulated protocol fees. |
| `selfDeprecated` | `SatoHook` | Indicates whether new buys are disabled. |
| `lastBuyBlock` | `SatoHook` | Tracks each swapper's last curve buy block. |

These variables define the observable monetary state of the protocol.

## Authorization Properties

### Token Authorization

`SatoToken` allows minting and burning only by the locked `minter`.

In the deployed protocol, the locked minter is `SatoHook`.

Therefore:

```text
caller != minter  =>  mint/burn reverts
caller == minter  =>  mint/burn may execute
```

The ERC-20 contract does not expose owner-controlled minting, administrative burning, pausing, blacklisting, or upgrade logic.

### Hook Authorization

`SatoHook` hook entry points are protected by the Uniswap v4 `PoolManager` execution context.

For callback-controlled functions:

```text
msg.sender != POOL_MANAGER  =>  revert
```

This restricts hook execution to the configured settlement layer.

### Router Role

`SatoSwapRouter` may route direct curve buys and sells by pre-settling input currency to the `PoolManager` and passing the swapper address through hook data.

The router does not own the reserve, does not control the curve, and does not authorize minting or burning independently.

## Buy Transition

A valid curve buy is an exact-input ETH -> SATO swap through the configured curve pool.

### Preconditions

A buy requires:

```text
pool == configured ETH/SATO curve pool
amountSpecified < 0
zeroForOne == true
ethIn <= 5 ETH
selfDeprecated == false
hookData contains swapper address
```

If these conditions are not satisfied, execution reverts.

### Transition

The buy transition can be summarized as:

```text
fee = ethIn * 30 / 10000
ethToCurve = ethIn - fee
fairSato = Curve.mintFor(ethCum, ethToCurve)
mintAmount = entropyAdjusted(fairSato)
```

Then:

```text
SatoToken.mint(PoolManager, mintAmount)
ethCum += ethToCurve
totalMintedFair += fairSato
feesAccrued += fee
lastBuyBlock[swapper] = block.number
```

ETH is taken into `SatoHook`.

SATO is routed to the buyer through `PoolManager` settlement.

### Postconditions

After a valid buy:

```text
ethCum increases by post-fee ETH
totalMintedFair increases by fair curve output
feesAccrued increases by buy fee
SATO supply increases by minted amount
hook ETH balance increases by ETH input
```

If the fair supply threshold is reached:

```text
totalMintedFair >= 99% of K_SUPPLY  =>  selfDeprecated = true
```

## Sell Transition

A valid curve sell is an exact-input SATO -> ETH swap through the configured curve pool.

### Preconditions

A sell requires:

```text
pool == configured ETH/SATO curve pool
amountSpecified < 0
zeroForOne == false
hookData contains swapper address
cooldown condition satisfied
hook ETH balance >= ethOut
```

If the seller attempts to sell in the same block as its last curve buy, execution reverts with:

```text
CooldownActive()
```

### Transition

The sell transition converts actual SATO input into fair-curve units:

```text
satoFairIn = satoIn * totalMintedFair / actualSupply
```

Then computes redemption:

```text
ethRaw = Curve.burnFor(totalMintedFair, satoFairIn)
fee = ethRaw * 30 / 10000
ethOut = ethRaw - fee
```

Then:

```text
SatoToken.burn(SatoHook, satoIn)
ethCum -= ethRaw
totalMintedFair -= satoFairIn
feesAccrued += fee
```

ETH is settled to the seller through the `PoolManager`.

### Postconditions

After a valid sell:

```text
SATO supply decreases by satoIn
totalMintedFair decreases by fair-curve input
feesAccrued increases by sell fee
hook ETH balance decreases by ethOut
```

If `ethRaw` exceeds `ethCum`, the implementation saturates:

```text
ethCum = 0
```

## Reserve Equation

The sell-side curve reserve is defined as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

This equation separates redemption backing from fee accounting.

A reviewer can inspect:

```text
hook ETH balance
feesAccrued
curveReserveEth()
```

to reason about reserve state.

## Curve Equations

The forward curve is:

```text
q(e) = K * (1 - e^(-e / S))
```

Where:

```text
K = 21,000,000 SATO
S = 500 ETH
e = cumulative ETH committed to the fair curve
```

The inverse redemption curve is:

```text
ethOutRaw = S * ln((K - q + b) / (K - q))
```

Where:

```text
q = current fair-curve supply
b = fair-curve SATO amount being redeemed
```

The marginal price is:

```text
p(e) = S * e^(e / S) / K
```

These formulas are implemented in the `Curve` library using PRBMath `UD60x18` fixed-point arithmetic.

## Fair Supply Reasoning

`totalMintedFair` is the canonical supply used for curve accounting.

Actual ERC-20 `totalSupply` may differ from `totalMintedFair` because the first `100` blocks included an entropy adjustment to actual buy output.

For sells, the protocol converts actual SATO input into fair units:

```text
satoFairIn = satoIn * totalMintedFair / actualSupply
```

This keeps inverse-curve redemption tied to the canonical curve position rather than raw ERC-20 supply.

## Safety Properties

The protocol is designed to preserve the following safety properties across valid execution:

| Property | Reasoning |
| --- | --- |
| No owner-controlled minting | `mint()` is restricted to locked `SatoHook`. |
| No administrative burning | `burn()` is restricted to locked `SatoHook`. |
| No reserve withdrawal function | `SatoHook` exposes no owner or treasury withdrawal path. |
| No external curve liquidity | `beforeAddLiquidity` always reverts. |
| Exact-input curve execution | Exact-output swaps revert. |
| Pool configuration validation | Hook accepts only the configured ETH/SATO curve pool. |
| Reserve-backed sells | Sell execution checks available ETH before settlement. |
| Public accounting | Monetary state is readable on-chain. |

## Liveness and Limits

The protocol is not designed to guarantee that every desired action can always succeed.

Important limits include:

- buys stop after self-deprecation
- sells require sufficient ETH reserve
- exact-output swaps are unsupported
- router behavior must satisfy PoolManager settlement requirements
- secondary markets may offer better or worse pricing than the curve

These limits are part of the protocol design and should not be interpreted as price guarantees.

## Secondary Market Separation

Secondary AMM trades do not use the bonding curve unless the curve pool is explicitly selected.

A secondary trade does not update:

```text
ethCum
totalMintedFair
curveReserveEth
SATO totalSupply
```

Only curve-routed buys and sells change protocol monetary state.

## Public Verifiability

The reasoning model can be checked against on-chain state.

Reviewers can inspect:

- `SatoToken.minter()`
- `SatoToken.totalSupply()`
- `SatoHook.ethCum()`
- `SatoHook.totalMintedFair()`
- `SatoHook.feesAccrued()`
- `SatoHook.curveReserveEth()`
- hook ETH balance
- `SatoSwapRouter` transactions
- mint and burn events
- PoolManager settlement transactions

## Conclusion

SATO's formal reasoning framework is based on restricted authorization, deterministic curve math, explicit state transitions, on-chain reserve custody, and public verifiability.

This document does not prove correctness. It describes the logical structure reviewers can use to reason about protocol behavior from verified source code and on-chain state.

## Related Documentation

- `docs/protocol/03_ERC20.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/research/02_Protocol_Invariants.md`
- `docs/security/03_Trust_Model.md`
