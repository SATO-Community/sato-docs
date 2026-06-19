# Security Guarantees

## Overview

This document summarizes the security properties intended by the deployed SATO protocol architecture.

The word "guarantees" in this document refers to properties enforced by the verified deployed contracts under normal Ethereum execution assumptions.

This document is not an audit report, formal verification report, or statement that the protocol is risk-free.

## Scope

The properties below apply to the documented deployed contracts:

| Component | Address | Role |
| --- | --- | --- |
| `SatoToken` | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` | ERC-20 accounting and locked mint/burn authorization |
| `SatoHook` | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` | Monetary execution, reserve custody, curve settlement |
| `SatoSwapRouter` | `0x06A645079cd4F3Bb38FfaD47f92180B8041145E3` | Direct curve buy/sell routing |
| Uniswap v4 `PoolManager` | `0x000000000004444c5dc75cB358380D2e3dE08A90` | Settlement layer |

## Guarantee 1 — Locked Mint Authority

`SatoToken.mint()` can only be called by the locked `minter`.

In the deployed protocol, the locked minter is:

```text
0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888
```

This is the deployed `SatoHook`.

There is no owner-controlled mint function.

## Guarantee 2 — Locked Burn Authority

`SatoToken.burn()` can only be called by the locked `minter`.

In the deployed protocol, this means burn execution is restricted to `SatoHook`.

Burning occurs as part of curve redemption rather than administrator-controlled balance editing.

## Guarantee 3 — One-Time Minter Assignment

`SatoToken.setMinter()` can set the minter only once.

After the minter is assigned, it cannot be changed.

This permanently links ERC-20 mint/burn authority to the deployed protocol hook.

## Guarantee 4 — No Token Owner Controls

The SATO ERC-20 contract does not implement:

- owner role
- pause function
- blacklist function
- whitelist function
- transfer tax
- account freeze function
- proxy upgrade path
- arbitrary balance editing function

Standard ERC-20 transfers, approvals, allowances, and balance queries remain available.

## Guarantee 5 — Curve Pool Validation

`SatoHook` accepts only the configured ETH/SATO curve pool.

The hook validates:

- native ETH as `currency0`
- SATO as `currency1`
- configured pool fee
- deployed `SatoHook` as hook

Unsupported pool configurations revert.

## Guarantee 6 — External Curve Liquidity Rejection

The curve pool does not accept ordinary external AMM liquidity.

`beforeAddLiquidity` always reverts with:

```text
LiquidityAdditionsForbidden()
```

This keeps curve execution controlled by `SatoHook` rather than third-party LP positions.

## Guarantee 7 — Exact-Input Curve Execution

`SatoHook` supports exact-input curve swaps.

Exact-output swaps revert with:

```text
ExactOutputUnsupported()
```

This keeps buy and sell execution based on known input amounts.

## Guarantee 8 — Deterministic Curve Math

Curve issuance and redemption use deterministic formulas implemented in the `Curve` library.

Forward issuance:

```text
q(e) = K * (1 - e^(-e / S))
```

Inverse redemption:

```text
ethOutRaw = S * ln((K - q + b) / (K - q))
```

Where:

```text
K = 21,000,000 SATO
S = 500 ETH
```

The curve uses PRBMath `UD60x18` fixed-point arithmetic.

## Guarantee 9 — On-Chain Reserve Custody

ETH reserves for curve redemption are held by `SatoHook`.

The hook does not expose an owner withdrawal function, governance withdrawal function, or treasury transfer function.

The sell-side reserve is exposed as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

## Guarantee 10 — Fee Accounting Separation

Protocol fees are tracked through:

```text
feesAccrued
```

The reserve view excludes accrued fees:

```text
curveReserveEth = hook ETH balance - feesAccrued
```

This separates redemption backing from accumulated fee accounting.

## Guarantee 11 — Same-Block Cooldown

The hook records each swapper's last curve buy block.

A wallet cannot sell through the curve in the same block as its last curve buy.

A violation reverts with:

```text
CooldownActive()
```

## Guarantee 12 — Maximum Single Buy

A single curve buy is capped at:

```text
5 ETH
```

A larger buy reverts with:

```text
BuyTooLarge()
```

This constrains single-transaction curve impact.

## Guarantee 13 — Self-Deprecation of New Buys

When fair curve supply reaches the exhaustion threshold:

```text
totalMintedFair >= 99% of K_SUPPLY
```

the hook sets:

```text
selfDeprecated = true
```

After that point:

- new buys revert
- sells remain available against existing curve reserves

## Guarantee 14 — Router Is Not Monetary Authority

`SatoSwapRouter` may route direct curve buys and sells.

The router does not:

- hold the ETH reserve
- define curve math
- authorize minting independently
- authorize burning independently
- change protocol state outside valid routed execution

Monetary behavior remains enforced by `SatoHook`, `Curve`, and `SatoToken`.

## Guarantee 15 — Public Verifiability

The protocol exposes key state for public review.

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

## Non-Guarantees

SATO does not guarantee:

- market price
- stable price
- secondary market liquidity
- profitable minting
- profitable burning
- successful arbitrage
- absence of MEV
- correctness of third-party routers
- correctness of all frontends
- formal verification
- absence of all contract bugs

These remain outside the scope of protocol-level security properties.

## Assumptions

The above properties assume:

- Ethereum Mainnet executes correctly
- users interact with the documented deployed contracts
- Uniswap v4 `PoolManager` behaves according to its design
- routers and frontends construct intended transactions
- users review transaction routes and approvals

## Summary

SATO's security properties are centered on locked mint/burn authority, deterministic curve execution, on-chain reserve custody, external liquidity rejection, fee accounting separation, and public verifiability.

These properties reduce discretionary administrative risk, but they do not remove market risk, smart contract risk, router risk, or user execution risk.

## Related Documentation

- `docs/security/02_Threat_Model.md`
- `docs/security/03_Trust_Model.md`
- `docs/security/05_False_Positive_Analysis.md`
- `docs/security/06_Scanner_Compatibility.md`
- `docs/protocol/03_ERC20.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/06_Reserve_Model.md`
