# Forking and Independent Implementations

## Purpose

This guide is for developers who want to study SATO's architecture or build an independent protocol inspired by its public behavior.

SATO's deployed contracts are publicly readable and exact-match verified. That makes the implementation inspectable and reproducible at the bytecode level. It does not automatically make every source file freely reusable under an open-source license.

## Source and License Boundary

The deployed SATO contracts currently show `-NA-` in Etherscan's license field:

| Component | Verified Source |
| --- | --- |
| SatoToken | [Etherscan](https://etherscan.io/address/0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09#code) |
| SatoHook | [Etherscan](https://etherscan.io/address/0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888#code) |
| SatoSwapRouter | [Etherscan](https://etherscan.io/address/0x06A645079cd4F3Bb38FfaD47f92180B8041145E3#code) |
| SatoHook Sourcify package | [Exact-match repository](https://repo.sourcify.dev/contracts/full_match/1/0x0000f07d2b5f1ddf3244b8780f972f306efd2888/) |

Therefore:

- source availability permits inspection and verification
- this documentation repository does not grant a license to copy the original contract source
- builders should obtain permission from the relevant copyright holder or write an independent implementation from the documented behavior and public interfaces
- third-party dependency licenses must be reviewed separately

The MIT License in this repository applies to the community-authored documentation, not automatically to the deployed contract source packages.

## Brand Boundary

An independent implementation should use its own:

- project name and token symbol
- visual identity and logo
- websites and community channels
- contract addresses and deployment records

Do not describe an independent deployment as SATO, an official SATO migration, or an upgrade to SATO unless that statement can be independently established. A fork cannot modify the already deployed SATO contracts.

## Architecture to Understand

Review the system as a group rather than copying only the ERC-20 token:

1. `SatoToken` manages balances, total supply, and the locked protocol minter relationship.
2. `SatoHook` implements curve pricing, mint and burn behavior, reserve accounting, fees, limits, and Uniswap v4 callbacks.
3. `SatoSwapRouter` provides a compatible direct execution path through `PoolManager`.
4. Uniswap v4 `PoolManager` provides settlement and invokes the hook according to the pool key.

Important state relationships include:

- `ethCum`
- `totalMintedFair`
- ERC-20 `totalSupply()`
- `feesAccrued`
- `curveReserveEth()`
- `selfDeprecated`

An independent implementation must define and test how these values move together on every buy, sell, mint, burn, fee, and direct ETH transfer path.

## Parameters That Require an Explicit Design Decision

Do not inherit economic constants without analysis. At minimum, review:

| Area | Examples |
| --- | --- |
| Curve | supply asymptote, scale, precision and saturation behavior |
| Issuance | buy cap, output calculation, exhaustion threshold |
| Redemption | inverse calculation, reserve checks, fee treatment |
| Supply accounting | fair supply versus actual ERC-20 supply |
| Execution protection | cooldown, exact-input/output support, slippage handling |
| Launch behavior | entropy or other temporary output rules |
| Reserve | reserve asset, custody, accounting, direct-transfer behavior |
| Fees | rate, destination, withdrawal or non-withdrawal design |
| Pool | currencies, fee tier, tick spacing, hook permissions |
| Router | settlement flow, hook data, recipient handling |

Changing one parameter can change solvency, redemption behavior, reachable supply, and market incentives. Economic simulation should precede deployment.

## Uniswap v4 Hook Requirements

A v4 hook is not deployed like an ordinary contract.

Uniswap v4 encodes callback permissions in specific bits of the hook address. The chosen address must agree with `getHookPermissions()`; otherwise the intended callbacks will not execute correctly.

For a real deployment:

1. select the exact callbacks required
2. calculate the corresponding hook flags
3. pin the intended `PoolManager` and constructor arguments
4. mine a compatible CREATE2 salt and hook address
5. deploy with the same creation code and constructor arguments used during mining
6. verify the deployed address and permission bits before pool initialization

See the official [Uniswap v4 Hooks overview](https://developers.uniswap.org/docs/protocols/v4/concepts/hooks) and [Hook Deployment guide](https://developers.uniswap.org/docs/protocols/v4/guides/hooks/hook-deployment).

## Recommended Build Process

### 1. Reproduce Before Modifying

Reconstruct the documented formulas and state transitions in tests. Confirm that forward curve, inverse curve, fee accounting, reserve accounting, and supply conversion agree under rounding and boundary cases.

### 2. Pin the Toolchain

The deployed SATO components are verified with Solidity `0.8.26`, optimizer enabled with `200` runs, and Cancun EVM settings. An independent implementation may use a different toolchain, but every dependency and compiler setting should be pinned and documented.

### 3. Write Tests Before Deployment

Include:

- unit tests for every pricing and accounting branch
- fuzz tests for curve and inverse-curve round trips
- invariant tests for reserve and supply relationships
- mainnet-fork tests against the intended PoolManager
- tests for zero, maximum, and near-exhaustion states
- tests for direct ETH transfers and fee accounting
- slippage, rounding, and same-block execution tests
- simulations covering sustained buys, sustained sells, and mixed activity

### 4. Review Deployment Wiring

Confirm:

- token minter authority is assigned exactly once as intended
- the hook points to the correct token and PoolManager
- the router points to the correct hook and pool
- the pool key uses the intended currencies, fee, spacing, and hook
- there is no unintended owner, proxy, pause, or withdrawal path
- constructor arguments and deployed bytecode are publicly verifiable

### 5. Obtain Independent Review

A successful compilation is not a security review. Commission independent smart-contract and economic review before accepting material user funds.

### 6. Publish Verification Material

Publish:

- source and explicit license
- compiler and dependency lock information
- deployment scripts
- test and invariant results
- contract addresses and constructor arguments
- pool ID and pool key
- known limitations and trust assumptions
- audit reports and unresolved findings

## What Is Not Included

This repository does not publish the source, deployment configuration, API internals, or credentials of the hosted SATO community dashboard. The dashboard is an independent interface over public information and is not required to inspect or independently implement the protocol contracts.

## Safety Notice

Do not deploy a copy-pasted monetary contract with real funds.

A small change in curve math, fixed-point rounding, hook permissions, settlement deltas, reserve accounting, or token authorization can create irreversible loss. Use test environments, reproducible builds, independent review, and a distinct identity for every independent implementation.

## Related Reading

- [Protocol Overview](../protocol/01_Protocol_Overview.md)
- [Architecture](../protocol/02_Architecture.md)
- [ERC-20 Token](../protocol/03_ERC20.md)
- [SatoHook](../protocol/04_SatoHook.md)
- [Bonding Curve](../protocol/05_Bonding_Curve.md)
- [Reserve Model](../protocol/06_Reserve_Model.md)
- [Protocol Invariants](../research/02_Protocol_Invariants.md)
- [Public Review Checklist](../security/07_Public_Review_Checklist.md)

