# Security Overview

## Overview

This section documents the security architecture, trust assumptions, threat model, and scanner interpretation context for the SATO protocol.

SATO is designed as an operator-free monetary protocol where issuance, redemption, reserve accounting, and token supply changes are enforced by deployed smart contracts rather than discretionary administrators.

This documentation is not an audit report.

It is a technical reference intended to help reviewers understand how the verified contracts are structured and how common scanner findings should be interpreted in protocol context.

## Security Scope

The security documentation focuses on the following deployed components:

| Component | Address | Role |
| --- | --- | --- |
| `SatoToken` | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` | ERC-20 accounting and locked mint/burn authorization |
| `SatoHook` | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` | Monetary execution, curve settlement, reserve custody |
| `SatoSwapRouter` | `0x06A645079cd4F3Bb38FfaD47f92180B8041145E3` | Direct curve buy/sell routing |
| Uniswap v4 `PoolManager` | `0x000000000004444c5dc75cB358380D2e3dE08A90` | Settlement layer |

The core monetary authority is `SatoHook`.

`SatoSwapRouter` is an execution router. It does not hold the reserve and does not define monetary policy.

## Security Posture

SATO minimizes discretionary control by removing common administrative mechanisms.

The deployed protocol is designed without:

- owner-controlled minting
- owner-controlled reserve withdrawal
- upgradeable proxy logic
- pause controls
- blacklist controls
- whitelist controls
- transfer taxes
- treasury-directed issuance

Minting and burning exist, but they are restricted to the locked `SatoHook` contract and are part of curve buy/sell execution.

## Reserve Security

ETH reserve custody is held by `SatoHook`.

The sell-side curve reserve is exposed as:

```text
curveReserveEth = address(SatoHook).balance - feesAccrued
```

The hook does not expose an owner withdrawal function, governance withdrawal function, or treasury transfer function.

ETH leaves the hook through curve sell execution when SATO is burned and ETH is settled through the Uniswap v4 `PoolManager`.

## Scanner Context

Automated scanners may identify mint and burn functions in the ERC-20 contract.

Viewed in isolation, these functions may appear as privileged balance-changing functionality.

In the full protocol architecture:

- the ERC-20 minter is permanently locked
- the locked minter is `SatoHook`
- minting occurs during valid curve buys
- burning occurs during valid curve sells
- reserve accounting is handled by the hook
- monetary behavior follows verified contract logic

Scanner output should therefore be interpreted together with the verified source code and protocol architecture.

## Documentation Contents

| Document | Purpose |
| --- | --- |
| `02_Threat_Model.md` | Describes threat categories, mitigations, and out-of-scope risks. |
| `03_Trust_Model.md` | Explains what participants must trust and what the protocol removes from trust assumptions. |
| `04_Security_Guarantees.md` | Summarizes security properties intended by the deployed architecture. |
| `05_False_Positive_Analysis.md` | Explains why some automated scanner findings may require protocol-level context. |
| `06_Scanner_Compatibility.md` | Provides guidance for interpreting SATO through token scanners and listing review tools. |

## Review Notes

Security reviewers should inspect:

- verified source code
- `SatoToken.minter()`
- `SatoHook.ethCum()`
- `SatoHook.totalMintedFair()`
- `SatoHook.feesAccrued()`
- `SatoHook.curveReserveEth()`
- hook ETH balance
- `SatoSwapRouter` transactions
- mint and burn events
- PoolManager settlement transactions

## Limitations

This documentation does not claim that the protocol is risk-free.

It does not guarantee:

- market price
- secondary market liquidity
- profitable trading
- resistance to all forms of MEV
- correctness of third-party routers
- correctness of all external integrations
- formal verification

It describes the verified architecture and the trust assumptions that remain.

## Related Documentation

- `docs/protocol/03_ERC20.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/research/02_Protocol_Invariants.md`
- `docs/research/05_Formal_Reasoning.md`
