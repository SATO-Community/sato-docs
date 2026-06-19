# Bonding Curve

## Overview

SATO issuance and redemption are governed by a deterministic bonding curve.

The curve defines how much SATO is minted for ETH committed to the protocol and how much ETH is redeemed when SATO is burned through the curve pool.

The math is implemented in the internal `Curve` library and executed by `SatoHook`.

## Curve Parameters

| Parameter | Value | Purpose |
| --- | --- | --- |
| `K_SUPPLY` | `21,000,000e18` | Asymptotic fair supply of the curve. |
| `S` | `500e18` | Curve scale parameter, denominated in ETH. |
| `MAX_EXP_X` | `50e18` | Saturation bound for exponential calculations. |

`K_SUPPLY` is the asymptotic supply target of the fair curve. The curve approaches this value but does not treat it as an ordinary fixed mint cap.

`S` controls how quickly the curve advances as ETH enters the system.

## Forward Curve

The forward curve computes cumulative fair SATO minted after cumulative ETH input.

```text
totalMinted(eth) = K * (1 - e^(-eth / S))
```

Where:

```text
K = 21,000,000 SATO
S = 500 ETH
eth = cumulative ETH committed to the fair curve
```

This curve is monotonic. As cumulative ETH increases, fair minted supply increases toward `K_SUPPLY`.

## Mint Calculation

A buy does not mint based on total ETH alone. It mints the difference between the new curve position and the previous curve position.

```text
mintFor(ethBefore, ethIn)
  = totalMinted(ethBefore + ethIn) - totalMinted(ethBefore)
```

In `SatoHook`, the protocol fee is removed before ETH advances the curve.

```text
fee = ethIn * 30 / 10000
ethToCurve = ethIn - fee
fairSato = Curve.mintFor(ethCum, ethToCurve)
```

The post-fee amount, `ethToCurve`, advances `ethCum`.

## Marginal Price

The marginal curve price is the ETH cost per SATO at a given curve position.

```text
marginalPrice(eth) = S * e^(eth / S) / K
```

As cumulative ETH increases, the marginal price increases exponentially.

This means each later unit of fair SATO becomes more expensive to mint than earlier units.

## Inverse Curve

The inverse curve calculates ETH redemption when SATO is sold through the curve pool.

For a burn of `satoIn` from the current fair supply position:

```text
burnFor(currentTotal, satoIn)
  = S * ln((K - currentTotal + satoIn) / (K - currentTotal))
```

Where:

```text
currentTotal = current fair-curve supply
satoIn = fair-curve amount being redeemed
```

The inverse curve moves the curve position backward and calculates the ETH owed before fees.

## ETH At Supply

The curve also defines the ETH amount corresponding to a fair supply position.

```text
ethAt(currentTotal)
  = S * ln(K / (K - currentTotal))
```

This is the inverse of the forward cumulative mint function.

## Fair Supply vs Actual Supply

`SatoHook` tracks `totalMintedFair` separately from the ERC-20 `totalSupply`.

This is necessary because the launch entropy mechanism could adjust actual minted output during the first `100` blocks after hook deployment.

During a sell, actual SATO input is converted into fair-curve units:

```text
satoFairIn = satoIn * totalMintedFair / actualSupply
```

This keeps redemption tied to the canonical fair curve position rather than raw token supply.

## Buy Execution and the Curve

During a buy:

```text
ETH input
  -> protocol fee removed
  -> remaining ETH advances ethCum
  -> Curve.mintFor calculates fair SATO output
  -> entropy may adjust actual minted amount
  -> SATO is minted through SatoToken
  -> ETH is held by SatoHook
```

Buy-side state updates include:

```text
ethCum += ethToCurve
totalMintedFair += fairSato
feesAccrued += fee
lastBuyBlock[swapper] = block.number
```

A single buy is capped at `5 ETH`.

Exact-output buys are not supported.

## Sell Execution and the Curve

During a sell:

```text
SATO input
  -> converted into fair-curve units
  -> Curve.burnFor calculates raw ETH redemption
  -> protocol fee removed
  -> SATO is burned through SatoToken
  -> ETH is settled back to the seller through PoolManager
```

Sell-side state updates include:

```text
ethCum -= ethRaw
totalMintedFair -= satoFairIn
feesAccrued += fee
```

If `ethRaw` exceeds `ethCum`, the hook saturates `ethCum` to zero.

A wallet cannot sell in the same block as its last curve buy.

## Protocol Fee

The protocol fee is applied outside the `Curve` library by `SatoHook`.

The fee is `0.3%`.

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

The sell-side reserve is exposed as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

## Entropy Window

During the first `100` blocks after hook deployment, the hook applies an entropy multiplier to buy output.

The multiplier range is:

```text
0.9x to 1.1x
```

The entropy multiplier affects actual minted SATO, but the canonical curve state advances by the fair curve amount.

After the entropy window, actual minted output equals fair curve output.

## Self-Deprecation Threshold

The hook disables new buys when fair curve supply reaches the exhaustion threshold.

```text
totalMintedFair >= 99% of K_SUPPLY
```

When this condition is met:

```text
selfDeprecated = true
```

After self-deprecation:

- new buys revert with `SelfDeprecatedNoBuys()`
- sells continue to be supported against available curve reserves

This allows the protocol to stop active issuance while preserving redemption behavior.

## Precision and Saturation

The curve uses PRBMath `UD60x18` fixed-point arithmetic.

The implementation treats the curve as saturated when:

```text
eth / S >= 50
```

At that point, the residual unminted supply is below the useful precision floor for the curve calculation, and `totalMinted()` returns `K_SUPPLY`.

## Curve Pool vs Secondary Markets

The bonding curve applies only when a trade routes through the ETH/SATO curve pool handled by `SatoHook`.

Direct curve interaction may be routed through `SatoSwapRouter` or any compatible router that satisfies the Uniswap v4 `PoolManager` settlement and hook-data requirements.

A trade through a secondary AMM pool does not use this curve.

Secondary trades:

- do not mint SATO
- do not burn SATO
- do not update `ethCum`
- do not update `totalMintedFair`
- do not change the curve reserve

Only curve-routed buys and sells change protocol supply and reserve state.

## Design Summary

The bonding curve is the monetary schedule of SATO.

It defines:

- how ETH input creates fair SATO supply
- how SATO burns redeem ETH
- how marginal mint cost increases over time
- how redemptions move the curve backward
- how issuance transitions toward dormancy

The curve is deterministic, on-chain, and executed by `SatoHook` without discretionary monetary administration.

## Related Documentation

- `docs/protocol/01_Protocol_Overview.md`
- `docs/protocol/02_Architecture.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/security/05_False_Positive_Analysis.md`
