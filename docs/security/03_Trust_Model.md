# Trust Model

## Overview

This document describes the trust assumptions of the SATO protocol.

SATO is designed to minimize discretionary trust in administrators, treasury managers, governance committees, and off-chain reserve operators.

It does not eliminate all trust assumptions.

Instead, it shifts trust toward verified smart contract execution, Ethereum consensus, Uniswap v4 settlement behavior, and user-selected execution routes.

This document is not an audit report.

## Core Trust Principle

SATO's trust model is based on reducing human discretion over monetary behavior.

Participants should not need to trust an operator to:

- mint tokens fairly
- manage reserve assets
- decide redemption policy
- pause or unpause the protocol
- blacklist users
- update monetary rules
- withdraw protocol reserves

The protocol attempts to encode monetary behavior directly into deployed contracts.

## Trust Removed or Minimized

### No Trusted Issuer

SATO does not rely on a discretionary issuer.

New SATO is minted through curve execution.

`SatoToken.mint()` is restricted to the locked `SatoHook` contract.

### No Trusted Reserve Manager

ETH reserve custody is held by `SatoHook`.

The hook does not expose an owner withdrawal function, governance withdrawal function, or treasury transfer function.

The sell-side reserve is exposed as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

### No Trusted Upgrade Administrator

The deployed core contracts do not expose upgrade paths or proxy implementation replacement logic.

Monetary behavior is defined by deployed verified source code.

### No Trusted Transfer Controller

`SatoToken` does not implement:

- pause controls
- blacklists
- whitelists
- transfer taxes
- account freezing
- administrative transfer restrictions

### No Trusted Liquidity Provider for the Curve Pool

The curve pool does not rely on ordinary external AMM liquidity.

External liquidity additions revert with:

```text
LiquidityAdditionsForbidden()
```

The hook acts as the counterparty for curve-routed swaps.

## Trust That Remains

SATO minimizes discretionary operator trust, but the following assumptions remain.

### Ethereum

Participants trust Ethereum Mainnet for:

- transaction ordering
- state execution
- consensus
- block production
- finality assumptions

A failure at the Ethereum layer can affect SATO.

### Verified Contract Correctness

Participants trust that the deployed contracts behave as intended.

This includes:

- `SatoToken`
- `SatoHook`
- `Curve`
- `SatoSwapRouter`
- integrated Uniswap v4 interfaces

This documentation does not replace independent smart contract review.

### Uniswap v4 PoolManager

SATO curve execution uses the Uniswap v4 `PoolManager` as its settlement layer.

Participants trust the configured `PoolManager` address and its settlement behavior.

### Router Behavior

Direct curve interaction may be routed through `SatoSwapRouter`.

The official `SatoSwapRouter` is verified and minimal, but users still need to verify the transaction route and calldata they approve.

Third-party routers or aggregators may introduce additional trust assumptions.

### Frontend and Interface Integrity

Users may interact through websites, wallets, aggregators, or custom scripts.

A malicious or incorrect frontend can present misleading quotes, route through the wrong pool, or construct unexpected transactions.

Users should verify transaction details when possible.

### Market Liquidity

The protocol does not guarantee secondary market liquidity.

Secondary market prices are determined by market participants, liquidity providers, routing, slippage, and volatility.

### User Route Selection

A user may choose between:

- curve mint/burn route
- secondary AMM route
- aggregator route

Different routes can produce different outcomes.

The protocol does not guarantee that the selected route is economically optimal.

## Contract-Level Trust Boundaries

### SatoToken

Trust boundary:

```text
ERC-20 accounting and locked mint/burn authorization
```

Important checks:

- `minter` is locked to `SatoHook`
- no owner mint path
- no pause path
- no blacklist path
- no transfer tax path

### SatoHook

Trust boundary:

```text
monetary execution, curve state, reserve custody
```

Important checks:

- only configured `PoolManager` can call hook entry points
- only configured ETH/SATO pool is accepted
- external liquidity additions are rejected
- curve buys and sells update `ethCum` and `totalMintedFair`
- `curveReserveEth()` excludes `feesAccrued`

### SatoSwapRouter

Trust boundary:

```text
direct curve routing helper
```

Important checks:

- router is not the reserve holder
- router does not define curve math
- router does not authorize minting or burning independently
- router routes through `PoolManager` and passes hook data to `SatoHook`

### Secondary Markets

Trust boundary:

```text
ordinary market trading
```

Secondary market trades do not mint SATO, burn SATO, or update curve reserve state.

They only transfer existing SATO between market participants.

## What Users Do Not Need to Trust

Users do not need to trust a SATO operator to:

- maintain the reserve manually
- approve redemptions
- decide who can mint
- decide who can sell
- adjust the bonding curve
- upgrade token logic
- pause transfers
- blacklist addresses
- withdraw reserve ETH

These controls are not exposed by the deployed core contracts.

## What Users Still Need to Verify

Users should verify:

- they are interacting with the documented contract addresses
- the route is the intended curve or secondary route
- transaction calldata matches expected action
- slippage and price impact are acceptable
- wallet approvals are appropriate
- frontend links are authentic
- scanner output is interpreted with protocol context

## Trust Model Summary

SATO reduces trust in operators by encoding monetary behavior into deployed contracts.

The protocol removes or minimizes trusted roles related to issuance, reserve custody, upgrades, transfer restrictions, and administrative intervention.

Remaining trust assumptions are technical and market-based: Ethereum execution, deployed contract correctness, Uniswap v4 settlement behavior, router usage, frontend integrity, and secondary liquidity.

## Related Documentation

- `docs/security/02_Threat_Model.md`
- `docs/security/04_Security_Guarantees.md`
- `docs/security/05_False_Positive_Analysis.md`
- `docs/security/06_Scanner_Compatibility.md`
- `docs/protocol/03_ERC20.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/06_Reserve_Model.md`
