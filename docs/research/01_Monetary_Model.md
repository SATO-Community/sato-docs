# Monetary Model

## Overview

SATO implements a deterministic monetary model enforced by immutable smart contracts on Ethereum.

The protocol does not rely on discretionary issuance, treasury-controlled supply expansion, governance-directed monetary policy, or off-chain reserve management.

Instead, SATO supply evolves through two protocol actions:

- minting through the bonding curve
- burning through the inverse curve

The contract is the issuer.

## Core Principle

SATO is built around a simple monetary principle:

```text
Monetary policy should be enforced by code rather than by people.
```

No participant has discretionary authority to expand supply, redirect reserves, pause monetary execution, or replace the issuance mechanism.

## No Premine or Treasury Issuance

The SATO monetary model does not depend on an initial privileged allocation.

The protocol is designed without:

- premine
- team allocation
- foundation allocation
- insider round
- treasury-controlled issuance
- discretionary supply expansion

SATO enters circulation through protocol minting rather than administrative distribution.

## Issuance Model

New SATO is minted when ETH is committed through the curve pool.

Direct curve buys may be routed through `SatoSwapRouter` or any compatible router that satisfies the Uniswap v4 `PoolManager` settlement and hook-data requirements.

The buy-side flow is:

```text
ETH input
  -> protocol fee separated
  -> post-fee ETH advances the curve
  -> fair SATO output is calculated
  -> SATO is minted through SatoToken
  -> ETH is held by SatoHook
```

Issuance is calculated by the deterministic forward curve:

```text
q(e) = K * (1 - e^(-e / S))
```

Where:

```text
K = 21,000,000 SATO
S = 500 ETH
e = cumulative ETH committed to the fair curve
```

A buy mints the difference between the new and previous curve positions:

```text
minted = q(e_before + eth_to_curve) - q(e_before)
```

The protocol fee is applied before ETH advances the curve:

```text
fee = ethIn * 30 / 10000
ethToCurve = ethIn - fee
```

A single curve buy is capped at `5 ETH`.

## Redemption Model

SATO can be redeemed by selling through the curve pool.

Direct curve sells may also be routed through `SatoSwapRouter` or a compatible router. The router does not control redemption math or reserve custody.

The sell-side flow is:

```text
SATO input
  -> converted into fair-curve units
  -> inverse curve calculates raw ETH redemption
  -> protocol fee separated
  -> SATO is burned through SatoToken
  -> ETH is settled to the seller
```

The inverse redemption formula is:

```text
ethOutRaw = S * ln((K - q + b) / (K - q))
```

Where:

```text
q = current fair-curve supply
b = fair-curve SATO amount being redeemed
```

The sell fee is then applied:

```text
fee = ethOutRaw * 30 / 10000
ethOut = ethOutRaw - fee
```

## Reserve Model

ETH committed through curve buys is held by the `SatoHook` contract.

The curve reserve is exposed as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

The reserve is not an off-chain account, treasury, or multisig. It is ETH held by the deployed hook contract.

ETH leaves the reserve only through curve sell execution, when SATO is burned and ETH is settled through the Uniswap v4 `PoolManager`.

## Fee Model

The protocol applies a `0.3%` fee on both curve buys and curve sells.

On buys, the fee is separated before the curve advances.

On sells, the fee is separated from raw inverse-curve redemption.

Fees are tracked in:

```text
feesAccrued
```

The whitepaper describes these fees as accumulating inside the hook rather than functioning as a withdrawable treasury.

## Fair Supply and Actual Supply

`SatoHook` tracks canonical curve supply through:

```text
totalMintedFair
```

This fair supply may differ from ERC-20 `totalSupply` because the launch entropy window could adjust actual minted output during the first `100` blocks after hook deployment.

During sells, actual SATO input is converted into fair-curve units:

```text
satoFairIn = satoIn * totalMintedFair / actualSupply
```

This keeps redemption tied to the canonical curve position rather than raw token supply.

## Entropy Window

During the first `100` blocks after deployment, buys could receive an entropy multiplier between:

```text
0.9x and 1.1x
```

The whitepaper states that this launch window has closed.

After that window, curve minting is fully deterministic.

## Supply Evolution

SATO supply changes only through protocol execution.

Supply increases when:

```text
ETH is committed through the curve pool and SATO is minted.
```

Supply decreases when:

```text
SATO is sold through the curve pool and burned.
```

Trades through secondary markets do not change SATO supply.

Secondary AMM trades only move existing SATO between market participants.

## Secondary Market Relationship

The curve pool and secondary markets serve different functions.

The curve pool is the canonical issuance and redemption mechanism.

Secondary pools provide ordinary market liquidity.

A trade through the curve pool may mint or burn SATO.

A trade through a secondary pool does not mint SATO, burn SATO, update `ethCum`, update `totalMintedFair`, or move ETH in or out of the curve reserve.

## Self-Deprecation

The curve disables new buys when fair curve supply reaches the exhaustion threshold:

```text
totalMintedFair >= 99% of K_SUPPLY
```

After this threshold is reached:

- new curve buys revert
- sells continue against available curve reserves

This allows issuance to transition toward dormancy while preserving redemption behavior.

## Monetary Guarantees

The monetary model is designed to provide the following properties:

- issuance follows deterministic curve math
- redemption follows deterministic inverse curve math
- ETH reserve accounting is on-chain
- minting is restricted to the locked protocol hook
- burning is restricted to the locked protocol hook
- no owner can mint outside protocol execution
- no administrator can withdraw curve reserves
- no governance vote can alter deployed monetary rules

## Limitations

This model does not guarantee a market price.

Secondary market prices may diverge from curve quotes.

The curve determines protocol issuance and redemption behavior. It does not force external markets to trade at the curve price.

Users should compare curve quotes and secondary market quotes before trading.

## Summary

SATO replaces discretionary monetary management with deterministic smart contract execution.

The monetary model is built around curve-based issuance, inverse-curve redemption, on-chain ETH reserves, and locked mint/burn authorization.

Supply expands only when ETH is committed through the curve.

Supply contracts only when SATO is burned through the curve.

The protocol's monetary behavior is enforced by deployed contract logic rather than by an operator, treasury, or governance process.

## Related Documentation

- `docs/protocol/01_Protocol_Overview.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/research/02_Protocol_Invariants.md`
- `docs/security/03_Trust_Model.md`
