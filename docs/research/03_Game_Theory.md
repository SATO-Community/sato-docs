# Game Theory

## Overview

SATO is an operator-free monetary protocol where participant incentives are shaped by deterministic smart contract execution.

The protocol does not rely on administrators, governance committees, treasury managers, or discretionary monetary policy to coordinate behavior. Instead, participants interact with fixed rules deployed on Ethereum.

This document describes incentive structure and mechanism design. It does not claim price stability or guaranteed market outcomes.

## Core Incentive Model

SATO has two primary protocol actions:

- minting through the bonding curve
- burning through the inverse curve

Participants choose whether to interact with the curve or with secondary markets.

The curve defines protocol issuance and redemption.

Secondary markets define external market pricing.

## Rule Symmetry

The same protocol rules apply to all participants.

The protocol does not provide:

- privileged minting rights
- administrative exemptions
- special redemption rights
- governance-controlled monetary adjustments
- team or treasury issuance

Every curve participant interacts with the same deployed contract logic.

## Bonding Curve Incentives

The forward curve makes later issuance more expensive than earlier issuance.

The marginal mint price increases as cumulative ETH committed to the curve increases.

This creates a predictable issuance schedule:

```text
early curve mints occur at lower curve positions
later curve mints occur at higher curve positions
```

However, this does not guarantee market profit. Secondary market prices may trade above or below the curve price.

Participants must compare curve quotes with secondary market quotes before acting.

## Redemption Incentives

Selling through the curve burns SATO and redeems ETH according to the inverse curve.

The curve can function as an algorithmic redemption path when secondary market bids are weak.

However, the curve does not guarantee the best exit price.

A secondary market may offer a better price than curve redemption, especially when ordinary liquidity is deep.

The rational participant compares:

```text
curve burn quote
secondary market quote
fees and slippage
execution route
```

before choosing a trading path.

## Secondary Market Relationship

SATO distinguishes between two trading regimes:

| Route | Effect |
| --- | --- |
| Curve pool | May mint or burn SATO and update reserve state. |
| Secondary pool | Trades existing SATO without changing supply or reserve state. |

A secondary trade does not change:

- `ethCum`
- `totalMintedFair`
- curve reserve
- SATO total supply

A curve trade may change all of those values.

## Arbitrage Dynamics

When secondary market prices rise above the curve mint price, participants may be incentivized to mint through the curve and sell into secondary liquidity.

When secondary bids fall below the curve burn quote, participants may be incentivized to burn through the curve rather than sell on secondary markets.

These dynamics can create pressure between curve quotes and secondary market prices, but they do not force equality.

Market prices remain dependent on liquidity, routing, fees, volatility, and participant behavior.

## Fee Friction

The protocol applies a `0.3%` fee on both curve buys and curve sells.

Fees create friction against low-margin cycling behavior.

On buys:

```text
fee = ethIn * 30 / 10000
```

On sells:

```text
fee = ethRaw * 30 / 10000
```

Because fees apply in both directions, immediate buy/sell cycling is structurally less attractive than a frictionless curve.

## Maximum Buy Constraint

A single curve buy is capped at:

```text
5 ETH
```

This limits the ability of one transaction to consume a large portion of available curve issuance.

It does not prevent accumulation across multiple transactions, wallets, or time, but it constrains single-transaction curve impact.

## Cooldown Constraint

The hook enforces a short cooldown between a wallet's curve buy and first curve sell.

A wallet cannot sell through the curve in the same block as its last curve buy.

If it attempts to do so, execution reverts with:

```text
CooldownActive()
```

This raises the cost of same-block buy/sell strategies and flash-loan-style curve cycling.

## Launch Entropy

During the first `100` blocks after deployment, curve buys could receive an entropy multiplier between:

```text
0.9x and 1.1x
```

The multiplier was derived from previous block hash, swapper address, and ETH input amount.

This introduced uncertainty during the earliest launch window and made exact deployment-block optimization less predictable.

The whitepaper states that this window has closed. After that period, curve output is deterministic.

## Self-Deprecation

The hook disables new buys when fair curve supply reaches the exhaustion threshold:

```text
totalMintedFair >= 99% of K_SUPPLY
```

After self-deprecation:

- new buys revert
- sells continue against available curve reserves

This creates a transition from active issuance toward dormancy.

In this regime, secondary markets may become the primary trading venue, while the curve remains a redemption path.

## Trust Minimization

Participants do not need to model discretionary operator behavior for monetary execution.

There is no administrator who can:

- mint outside the curve
- withdraw the curve reserve
- change curve parameters
- pause curve execution
- redirect protocol fees
- blacklist a participant from ERC-20 transfers

The main trust assumptions shift toward:

- Ethereum execution
- deployed contract correctness
- router behavior
- PoolManager settlement behavior
- market liquidity

## Information Symmetry

Protocol state is publicly observable.

Participants can inspect:

- curve reserve
- hook ETH balance
- `feesAccrued`
- `ethCum`
- `totalMintedFair`
- SATO total supply
- curve mint and burn transactions
- secondary market liquidity

This transparency reduces reliance on private operator information.

It does not eliminate market risk or information differences between traders.

## Strategic Limitations

The protocol does not guarantee:

- stable price
- minimum market price
- secondary market liquidity
- profitable minting
- profitable burning
- arbitrage availability
- protection from volatility

The protocol defines rules. Market participants determine prices.

## Summary

SATO's game-theoretic design centers on deterministic issuance, inverse-curve redemption, fee friction, maximum buy constraints, cooldown enforcement, launch entropy, and separation between curve and secondary markets.

Participants interact with transparent rules rather than discretionary operators.

The result is a monetary mechanism where strategy is shaped by public contract state, curve math, fees, liquidity, and market prices rather than administrative intervention.

## Related Documentation

- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/research/01_Monetary_Model.md`
- `docs/research/02_Protocol_Invariants.md`
- `docs/security/03_Trust_Model.md`
