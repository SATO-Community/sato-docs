# Threat Model

## Overview

This document describes the main threat categories considered for the SATO protocol and how the deployed architecture attempts to reduce or isolate them.

SATO is designed to minimize discretionary monetary control by enforcing issuance, redemption, reserve accounting, and token supply changes through verified smart contracts.

This document is not an audit report.

It describes threat assumptions, mitigations, and risks that remain.

## Scope

The threat model focuses on the deployed SATO protocol components:

| Component | Address | Role |
| --- | --- | --- |
| `SatoToken` | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` | ERC-20 accounting and locked mint/burn authorization |
| `SatoHook` | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` | Monetary execution, reserve custody, curve settlement |
| `SatoSwapRouter` | `0x06A645079cd4F3Bb38FfaD47f92180B8041145E3` | Direct curve buy/sell routing |
| Uniswap v4 `PoolManager` | `0x000000000004444c5dc75cB358380D2e3dE08A90` | Settlement layer |

## Threat: Owner-Controlled Minting

### Description

A common token risk is the existence of an owner or administrator who can mint additional tokens at will.

### SATO Mitigation

`SatoToken.mint()` can only be called by the locked `minter`.

The deployed minter is `SatoHook`.

The minter can be set once and cannot be changed afterward.

There is no owner-controlled mint function.

### Residual Risk

Minting still exists as a protocol function, so scanners may flag it. It must be interpreted in the context of the locked minter and curve execution model.

## Threat: Administrative Burning

### Description

Some token contracts allow an administrator to destroy user balances or arbitrarily reduce supply.

### SATO Mitigation

`SatoToken.burn()` can only be called by the locked `SatoHook`.

Burning occurs during curve sell execution, when SATO is redeemed through the inverse curve.

There is no general owner-controlled burn mechanism.

### Residual Risk

The burn function is present and may be flagged by automated scanners. Its control path should be reviewed through the locked minter relationship.

## Threat: Reserve Withdrawal by an Administrator

### Description

A reserve-backed protocol may be vulnerable if an administrator can withdraw reserve assets.

### SATO Mitigation

ETH reserves are held by `SatoHook`.

The hook does not expose an owner withdrawal function, governance withdrawal function, or treasury transfer function.

The sell-side reserve is exposed as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

ETH leaves the hook through curve sell execution when SATO is burned and ETH is settled through the Uniswap v4 `PoolManager`.

### Residual Risk

Sell execution still depends on available ETH balance and correct PoolManager settlement behavior.

## Threat: Upgrade or Proxy Replacement

### Description

Upgradeable contracts can allow protocol behavior to change after deployment.

### SATO Mitigation

The deployed core contracts do not expose proxy upgrade logic or governance-controlled implementation replacement.

Monetary behavior is defined by verified deployed contract code.

### Residual Risk

Users should verify they are interacting with the documented deployed contracts and not unrelated interfaces or copied contracts.

## Threat: Pause, Blacklist, or Transfer Restriction

### Description

Some ERC-20 contracts include pause controls, blacklists, whitelists, or account freezing mechanisms.

### SATO Mitigation

`SatoToken` is designed as a minimal ERC-20 accounting contract.

It does not implement:

- pause controls
- blacklists
- whitelists
- transfer taxes
- account freezing
- administrative transfer restrictions

### Residual Risk

Third-party venues, wallets, frontends, or interfaces may impose their own restrictions outside the SATO token contract.

## Threat: External Liquidity Manipulating Curve Execution

### Description

If ordinary AMM liquidity were allowed inside the curve pool, third-party liquidity could affect pricing and settlement assumptions.

### SATO Mitigation

The SATO curve pool forbids external liquidity additions.

`beforeAddLiquidity` always reverts with:

```text
LiquidityAdditionsForbidden()
```

The hook acts as the counterparty for curve-routed swaps.

### Residual Risk

Secondary markets may still exist separately and may have independent liquidity risk, price impact, or manipulation risk.

## Threat: Incorrect Pool Configuration

### Description

A hook may be misused if attached to an unintended pool or unsupported asset pair.

### SATO Mitigation

`SatoHook` validates the configured ETH/SATO pool.

It requires:

- native ETH as `currency0`
- SATO as `currency1`
- configured pool fee
- deployed `SatoHook` as hook

Unsupported configurations revert.

### Residual Risk

Users should verify the route and pool selected by frontends, routers, or aggregators.

## Threat: Same-Block Buy/Sell Cycling

### Description

A participant may attempt to buy and sell in the same block to exploit curve behavior or flash-loan-style execution.

### SATO Mitigation

`SatoHook` records each swapper's last curve buy block.

A sell in the same block as the wallet's last curve buy reverts with:

```text
CooldownActive()
```

### Residual Risk

This does not prevent all multi-wallet, multi-transaction, or cross-market strategies. It specifically constrains same-wallet same-block curve cycling.

## Threat: Single-Transaction Curve Impact

### Description

A large buy could significantly advance the curve in one transaction.

### SATO Mitigation

A single curve buy is capped at:

```text
5 ETH
```

This limits single-transaction curve impact.

### Residual Risk

The cap does not prevent accumulation through multiple transactions, wallets, or time.

## Threat: Launch-Block Optimization

### Description

At launch, bots may attempt to optimize for exact deployment-block curve conditions.

### SATO Mitigation

During the first `100` blocks after deployment, buys could receive an entropy multiplier between:

```text
0.9x and 1.1x
```

This introduced uncertainty into the earliest launch window.

The whitepaper states that this window has closed.

### Residual Risk

This mechanism applied only during the initial launch window and is no longer active.

## Threat: Price or Liquidity Misinterpretation

### Description

Users may confuse the protocol curve quote with a guaranteed market price.

### SATO Mitigation

SATO separates curve-routed issuance/redemption from secondary market trading.

A curve trade may mint or burn SATO.

A secondary market trade only transfers existing SATO and does not update reserve or supply state.

### Residual Risk

The protocol does not guarantee:

- market price
- secondary liquidity
- profitable minting
- profitable burning
- arbitrage availability
- low slippage
- price stability

## Threat: Router or Frontend Risk

### Description

Users may interact with the protocol through routers, aggregators, or frontends.

### SATO Mitigation

The official site uses `SatoSwapRouter`, a verified minimal router for direct curve buy/sell execution.

`SatoSwapRouter` does not hold the reserve and does not define monetary policy.

It routes curve interaction through the Uniswap v4 `PoolManager` and passes hook data to `SatoHook`.

### Residual Risk

Users must still trust or verify the interface, transaction calldata, route selection, and router behavior they use.

Compatible third-party routers may vary in behavior.

## Threat: PoolManager Dependency

### Description

SATO curve execution depends on Uniswap v4 `PoolManager` settlement behavior.

### SATO Mitigation

`SatoHook` validates that callbacks come from the configured `POOL_MANAGER`.

### Residual Risk

The protocol inherits trust assumptions from Uniswap v4 `PoolManager` behavior and settlement semantics.

## Threat: Automated Scanner Misclassification

### Description

Automated scanners may classify mint and burn functions as privileged or risky when viewed in isolation.

### SATO Mitigation

SATO documents the locked minter relationship:

```text
SatoToken.minter() == SatoHook
```

Minting and burning are part of curve execution rather than owner-controlled administration.

### Residual Risk

Scanner outputs may still require manual explanation during listings, integrations, or exchange reviews.

## Out-of-Scope Risks

The following risks are not eliminated by the SATO protocol design:

- Ethereum consensus failure
- bugs in deployed smart contracts
- bugs in Uniswap v4 `PoolManager`
- wallet compromise
- phishing or malicious frontends
- malicious third-party routers
- secondary market manipulation
- low liquidity
- MEV and transaction ordering effects
- user route-selection errors
- regulatory or off-chain operational risk

## Summary

SATO's threat model focuses on reducing discretionary monetary risk.

The protocol removes common administrative controls such as owner minting, reserve withdrawal, upgrades, pauses, blacklists, and transfer taxes.

Remaining risks are primarily related to deployed contract correctness, Ethereum execution, Uniswap v4 settlement behavior, router usage, secondary market liquidity, and user route selection.

## Related Documentation

- `docs/security/03_Trust_Model.md`
- `docs/security/04_Security_Guarantees.md`
- `docs/security/05_False_Positive_Analysis.md`
- `docs/security/06_Scanner_Compatibility.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/06_Reserve_Model.md`
