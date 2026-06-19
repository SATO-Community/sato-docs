# Economic Security

## Overview

Economic security describes how SATO's incentive design, curve mechanics, reserve accounting, and execution constraints work together to reduce discretionary monetary risk.

SATO does not rely on administrators, treasury managers, governance committees, or off-chain reserve operators to manage issuance and redemption.

Instead, economic behavior is enforced by deployed smart contracts.

This document describes economic security design. It does not claim price stability, market liquidity, or guaranteed trading outcomes.

## Security Objective

The core economic security objective is to minimize discretionary control over monetary state.

SATO is designed so that no privileged actor can:

- mint outside protocol execution
- withdraw curve reserves
- change the bonding curve
- pause monetary execution
- redirect protocol fees
- assign special redemption rights
- alter supply through governance

The protocol shifts economic trust from operators to verified contract logic.

## Deterministic Issuance

SATO issuance occurs through the bonding curve.

The forward curve is deterministic:

```text
q(e) = K * (1 - e^(-e / S))
```

Where:

```text
K = 21,000,000 SATO
S = 500 ETH
e = cumulative ETH committed to the fair curve
```

Because issuance follows this formula, participants can independently reproduce curve output from public state.

## Deterministic Redemption

SATO redemption occurs through the inverse curve.

The inverse curve calculates raw ETH owed for a burn:

```text
ethOutRaw = S * ln((K - q + b) / (K - q))
```

Where:

```text
q = current fair-curve supply
b = fair-curve SATO amount being redeemed
```

The hook burns SATO and settles ETH through the Uniswap v4 `PoolManager`.

Redemption is not controlled by an administrator.

## Reserve Integrity

ETH committed through curve buys is held by `SatoHook`.

The sell-side reserve is exposed as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

The hook does not expose an owner withdrawal function, governance withdrawal function, or treasury transfer function.

ETH leaves the hook through curve sell execution when SATO is burned and ETH is settled to the seller.

## Router Role

Direct curve buys and sells may be routed through `SatoSwapRouter`, a minimal verified router used by the official site.

`SatoSwapRouter` is not the reserve holder and does not define monetary policy.

The router helps pre-settle input currency to the Uniswap v4 `PoolManager` and passes the swapper address to `SatoHook` through hook data.

Minting, burning, reserve accounting, and curve math remain enforced by `SatoHook` and `Curve`.

## Fee Friction

SATO applies a `0.3%` fee on both curve buys and curve sells.

On buys:

```text
fee = ethIn * 30 / 10000
ethToCurve = ethIn - fee
```

On sells:

```text
fee = ethOutRaw * 30 / 10000
ethOut = ethOutRaw - fee
```

This fee friction increases the cost of immediate buy/sell cycling and low-margin curve abuse.

Fees are tracked in:

```text
feesAccrued
```

The whitepaper describes these fees as accumulating inside the hook rather than functioning as a withdrawable treasury.

## Maximum Buy Constraint

A single curve buy is capped at:

```text
5 ETH
```

This limits single-transaction impact on curve position and issuance.

It does not prevent accumulation over multiple transactions, wallets, or time. It is a constraint on transaction-level curve impact, not a complete anti-accumulation mechanism.

## Cooldown Constraint

The hook records each swapper's last curve buy block.

A wallet cannot sell through the curve in the same block as its last curve buy.

If it attempts to do so, the sell reverts with:

```text
CooldownActive()
```

This reduces same-block buy/sell cycling and raises the cost of flash-loan-style interaction with the curve.

## Launch Entropy

During the first `100` blocks after hook deployment, buy output could be adjusted by an entropy multiplier.

The multiplier range was:

```text
0.9x to 1.1x
```

The multiplier was derived from previous block hash, swapper address, and ETH input.

This made exact deployment-block optimization less predictable during launch.

The whitepaper states that this window has closed. After that period, buy output is deterministic.

## External Liquidity Isolation

The curve pool does not rely on ordinary external AMM liquidity.

External liquidity additions to the curve pool are forbidden.

`beforeAddLiquidity` always reverts with:

```text
LiquidityAdditionsForbidden()
```

This prevents third-party liquidity from changing the curve pool's monetary execution model.

Secondary AMM pools may exist separately, but they do not control protocol issuance or redemption.

## Secondary Market Risk

Secondary markets can trade above or below curve quotes.

The protocol does not guarantee:

- market price
- profitable minting
- profitable burning
- secondary liquidity
- arbitrage availability
- low slippage
- price stability

Participants must compare curve quotes, secondary market quotes, fees, routing, and slippage before trading.

The curve defines protocol issuance and redemption. It does not force external market prices.

## Self-Deprecation

The hook disables new buys when fair curve supply reaches the exhaustion threshold:

```text
totalMintedFair >= 99% of K_SUPPLY
```

After self-deprecation:

- new buys revert
- sells remain available against existing curve reserves

This reduces further issuance once the curve approaches exhaustion while preserving redemption behavior.

## Equal Rule Application

The protocol applies the same curve logic to all participants.

There are no:

- address-specific mint prices
- address-specific redemption rights
- admin exemptions
- whitelist-only curve access
- privileged user classes

Participants interact with the same deployed contract rules.

## Public Verifiability

Economic state can be reviewed on-chain.

Relevant values include:

- `SatoToken.totalSupply()`
- `SatoToken.minter()`
- `SatoHook.ethCum()`
- `SatoHook.totalMintedFair()`
- `SatoHook.feesAccrued()`
- `SatoHook.curveReserveEth()`
- hook ETH balance
- `SatoSwapRouter` transactions
- mint and burn events
- PoolManager settlement transactions

This transparency reduces reliance on private operator reporting.

## Remaining Trust Assumptions

SATO minimizes operator trust, but it does not eliminate all assumptions.

Important remaining assumptions include:

- Ethereum consensus and execution correctness
- deployed contract correctness
- Uniswap v4 `PoolManager` behavior
- router pre-settlement behavior
- curve math precision
- sufficient ETH reserve for redemptions
- user route selection
- secondary market liquidity

These assumptions should be considered when evaluating protocol risk.

## Security Perspective

Economic security in SATO comes from constraints and transparency rather than discretionary intervention.

The main constraints are:

- deterministic curve math
- locked minter path
- on-chain reserve custody
- fee friction
- maximum single-buy size
- cooldown enforcement
- external liquidity isolation
- self-deprecation of new buys

These mechanisms reduce specific classes of discretionary and short-term manipulation risk, but they do not remove market risk.

## Summary

SATO's economic security model is based on deterministic issuance, deterministic redemption, on-chain reserve custody, fee friction, transaction-level constraints, and public verification.

The protocol is designed to remove discretionary monetary control while making remaining risks visible through contract state and market behavior.

## Related Documentation

- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/research/01_Monetary_Model.md`
- `docs/research/02_Protocol_Invariants.md`
- `docs/research/03_Game_Theory.md`
- `docs/security/03_Trust_Model.md`
