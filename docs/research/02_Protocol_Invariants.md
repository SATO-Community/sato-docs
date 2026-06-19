# Protocol Invariants

## Overview

A protocol invariant is a property that should remain true across valid protocol execution.

SATO is designed around deterministic monetary behavior enforced by deployed smart contracts. The following invariants describe the core properties that the protocol architecture is intended to preserve.

This document describes protocol design properties. It is not an audit.

## Invariant 1 — Immutable Contract Logic

SATO monetary behavior is defined by deployed smart contracts.

The deployed `SatoToken` and `SatoHook` contracts do not expose upgrade functions, proxy upgrade paths, or governance-controlled replacement logic.

Once deployed, the monetary rules are enforced by the verified contract code.

## Invariant 2 — One-Time Minter Lock

`SatoToken` has one designated `minter`.

The minter can be set once through `setMinter(address newMinter)`.

After the minter is set, it cannot be changed.

In the deployed protocol, the locked minter is:

```text
0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888
```

This is the deployed `SatoHook` contract.

## Invariant 3 — No Owner-Controlled Issuance

SATO supply cannot be expanded by an owner wallet, treasury wallet, governance proposal, or administrator.

Minting is restricted to the locked `SatoHook` contract.

During valid curve buys, `SatoHook` calls:

```text
SatoToken.mint()
```

The ERC-20 contract does not decide when issuance occurs. It records issuance only when called by the locked protocol hook.

## Invariant 4 — No Administrative Burning

SATO cannot be arbitrarily burned by an administrator.

Burning is restricted to the locked `SatoHook` contract.

During valid curve sells, `SatoHook` calls:

```text
SatoToken.burn()
```

The burn path is part of curve redemption. It is not an owner-controlled balance editing mechanism.

## Invariant 5 — Curve-Routed Supply Changes

SATO supply changes only through curve-routed protocol execution.

Supply increases when ETH is committed through the curve pool and SATO is minted.

Supply decreases when SATO is sold through the curve pool and burned.

Secondary market trades do not mint or burn SATO.

A trade through a normal AMM pool only transfers existing SATO between participants.

## Invariant 6 — ETH Reserve Custody

Curve ETH reserves are held by the deployed `SatoHook` contract.

The hook does not expose an owner withdrawal function, governance withdrawal function, or treasury transfer function.

ETH enters the hook through curve buy execution.

ETH leaves the hook through curve sell execution when SATO is burned and ETH is settled through the Uniswap v4 `PoolManager`.

## Invariant 7 — Reserve View

The sell-side curve reserve is defined as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

This separates redemption backing from accumulated fee accounting.

The reserve can be reviewed on-chain through:

- hook ETH balance
- `feesAccrued`
- `curveReserveEth()`

## Invariant 8 — Deterministic Curve Accounting

The protocol tracks canonical curve state through:

```text
ethCum
totalMintedFair
```

`ethCum` tracks cumulative ETH committed to the fair curve after buy-side fees and after sell-side curve retractions.

`totalMintedFair` tracks canonical fair-curve supply.

Curve minting and redemption use deterministic formulas implemented in the `Curve` library.

## Invariant 9 — Fair Supply Conversion

Actual ERC-20 total supply may differ from fair curve supply because of the launch entropy window.

During sell execution, actual SATO input is converted into fair-curve units:

```text
satoFairIn = satoIn * totalMintedFair / actualSupply
```

This preserves redemption accounting against the canonical fair curve position.

## Invariant 10 — External Liquidity Is Forbidden in the Curve Pool

The SATO curve pool does not rely on ordinary external AMM liquidity.

`beforeAddLiquidity` always reverts with:

```text
LiquidityAdditionsForbidden()
```

The hook acts as the counterparty for curve-routed swaps.

## Invariant 11 — Exact-Input Curve Swaps

`SatoHook` supports exact-input curve swaps.

Exact-output swaps revert with:

```text
ExactOutputUnsupported()
```

This keeps curve execution based on known input amounts.

## Invariant 12 — Pool Configuration Validation

The hook accepts only the configured ETH/SATO curve pool.

During initialization and swap execution, the hook validates that the pool uses:

- native ETH as `currency0`
- SATO as `currency1`
- the configured `SatoHook`
- the configured pool fee

Unsupported pool configurations revert.

## Invariant 13 — Cooldown Enforcement

The hook records each swapper's last curve buy block.

A wallet cannot sell through the curve in the same block as its last curve buy.

If it attempts to do so, the sell reverts with:

```text
CooldownActive()
```

## Invariant 14 — Self-Deprecation of New Buys

When fair curve supply reaches the exhaustion threshold, new buys are disabled.

The threshold is:

```text
totalMintedFair >= 99% of K_SUPPLY
```

After self-deprecation:

- new buys revert
- sells remain available against existing curve reserves

## Invariant 15 — Public Verifiability

SATO protocol state is verifiable on Ethereum.

Reviewers can inspect:

- `SatoToken.totalSupply()`
- `SatoToken.minter()`
- `SatoHook.ethCum()`
- `SatoHook.totalMintedFair()`
- `SatoHook.feesAccrued()`
- `SatoHook.curveReserveEth()`
- hook ETH balance
- mint and burn events
- PoolManager settlement transactions

## Summary

SATO's protocol invariants are centered on deterministic execution, locked mint/burn authority, on-chain reserve custody, and transparent curve accounting.

The protocol is designed so that monetary state changes occur through verified contract logic rather than discretionary administration.

## Related Documentation

- `docs/protocol/03_ERC20.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/security/03_Trust_Model.md`
