# References

## Overview

This document lists primary references used for SATO protocol documentation.

References are grouped by source type: SATO protocol resources, verified contracts, Ethereum standards, Uniswap v4, libraries, and external scanner context.

This document is not an audit report.

## SATO Protocol Resources

### Official Website

https://sat0.org

Primary public interface for SATO protocol information and direct curve interaction.

### Whitepaper

https://sat0.org/whitepaper

Describes SATO's monetary model, reserve design, bonding curve, trading regimes, and operator-free design philosophy.

## Verified SATO Contracts

### SATO ERC-20

https://etherscan.io/token/0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09#code

Verified `SatoToken` contract on Ethereum Mainnet.

Role:

- ERC-20 token accounting
- locked minter authorization
- protocol-authorized mint and burn execution

### SatoHook

https://etherscan.io/address/0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888#code

Verified `SatoHook` contract on Ethereum Mainnet.

Role:

- Uniswap v4 hook
- curve buy and sell execution
- ETH reserve custody
- fee accounting
- protocol-authorized mint and burn coordination

### SatoSwapRouter

https://etherscan.io/address/0x06A645079cd4F3Bb38FfaD47f92180B8041145E3#code

Verified `SatoSwapRouter` contract on Ethereum Mainnet.

Role:

- direct curve buy/sell routing
- input pre-settlement to Uniswap v4 `PoolManager`
- hook-data forwarding to `SatoHook`

The router is not the reserve holder and does not define monetary policy.

### Sourcify Verified Source

https://repo.sourcify.dev/contracts/full_match/1/0x0000f07d2b5f1ddf3244b8780f972f306efd2888/

Full-match verified source package for `SatoHook`, including related source files such as `SatoToken.sol` and `Curve.sol`.

## Core Deployment Addresses

| Component | Address |
| --- | --- |
| SATO ERC-20 | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| SatoHook | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` |
| SatoSwapRouter | `0x06A645079cd4F3Bb38FfaD47f92180B8041145E3` |
| Uniswap v4 PoolManager | `0x000000000004444c5dc75cB358380D2e3dE08A90` |
| USDT | `0xdAC17F958D2ee523a2206206994597C13D831ec7` |

## Ethereum

### Ethereum Whitepaper

https://ethereum.org/en/whitepaper/

Introduces Ethereum and programmable smart contracts.

### Ethereum Developer Documentation

https://ethereum.org/en/developers/docs/

Official Ethereum documentation covering accounts, transactions, the EVM, and smart contract execution.

### Ethereum Mainnet Explorer

https://etherscan.io/

Public blockchain explorer used to inspect contract source code, token supply, transactions, logs, and on-chain state.

## ERC-20

### ERC-20 Token Standard

https://eips.ethereum.org/EIPS/eip-20

Token interface standard implemented by the SATO ERC-20 contract.

## Uniswap v4

### Uniswap v4 Documentation

https://docs.uniswap.org/contracts/v4/overview

Documentation for Uniswap v4 architecture, PoolManager, singleton design, and hooks.

### Uniswap v4 Whitepaper

https://uniswap.org/whitepaper-v4.pdf

Technical paper describing Uniswap v4 architecture and custom hooks.

### Uniswap App

https://app.uniswap.org/

Public interface for secondary market trading routes.

The official SATO website links secondary SATO/USDT trading through Uniswap.

## Smart Contract Libraries

### OpenZeppelin Contracts

https://github.com/OpenZeppelin/openzeppelin-contracts

Smart contract library used by `SatoToken` for ERC-20 accounting behavior.

### PRBMath

https://github.com/PaulRBerg/prb-math

Fixed-point math library used by `Curve` for exponential and logarithmic curve calculations.

## Scanner Context

Automated scanners may classify mint and burn functions as privileged functionality when viewed in isolation.

SATO documentation should be read together with the verified contracts, especially the locked minter relationship between `SatoToken` and `SatoHook`.

### GoPlus Security

https://gopluslabs.io/

Public token and smart contract risk analysis platform.

### TokenSniffer

https://tokensniffer.com/

Automated token analysis platform.

Scanner output should be interpreted together with source-level protocol architecture, not as a substitute for manual contract review.

## Related Repository Documentation

- `docs/protocol/01_Protocol_Overview.md`
- `docs/protocol/02_Architecture.md`
- `docs/protocol/03_ERC20.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/research/01_Monetary_Model.md`
- `docs/research/02_Protocol_Invariants.md`
- `docs/research/03_Game_Theory.md`
- `docs/research/04_Economic_Security.md`
- `docs/research/05_Formal_Reasoning.md`
- `docs/security/03_Trust_Model.md`
- `docs/security/05_False_Positive_Analysis.md`
