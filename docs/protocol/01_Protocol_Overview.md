# Protocol Overview

SATO is an Ethereum-native ERC-20 monetary protocol whose issuance and redemption are enforced by immutable smart contracts.

The protocol has no premine, no team treasury, no foundation allocation, no upgrade path, no pause function, and no discretionary issuer. Monetary behavior is defined by contract logic.

## Core Idea

SATO treats the contract as the issuer.

New SATO is minted only when ETH is committed through the bonding curve. SATO is burned when holders redeem through the inverse curve. The reserve is held on-chain by the hook contract and can be inspected publicly.

## Deployed Contracts

| Component | Address |
| --- | --- |
| SATO ERC-20 | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| SatoHook | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` |
| Uniswap v4 PoolManager | `0x000000000004444c5dc75cB358380D2e3dE08A90` |

Genesis block: `25,015,094`

## Architecture

SATO separates token accounting from monetary execution.

| Component | Role |
| --- | --- |
| `SatoToken` | ERC-20 balances, transfers, supply, mint, burn |
| `SatoHook` | Uniswap v4 hook that executes curve minting and burning |
| `Curve` | Bonding curve and inverse redemption math |

`SatoToken` has one locked minter. The minter is the protocol hook. Once set, the minter cannot be changed.

## Issuance

SATO issuance happens through the curve.

A buy routes ETH into the hook. The hook applies the forward curve, mints SATO, and advances the cumulative curve position.

Locked curve parameters include:

| Parameter | Value |
| --- | --- |
| Asymptotic supply | `21,000,000 SATO` |
| Curve scale | `500 ETH` |
| Maximum single mint | `5 ETH` |
| Protocol fee | `0.3%` |
| Cooldown | `1 block` |
| Initial entropy window | `100 blocks` |

## Redemption

A sell through the curve burns SATO and redeems ETH from the hook reserve using the inverse curve.

The curve reserve is the hook contract balance minus accrued protocol fees. The reserve moves out only through burn/redemption execution.

## Trading Regimes

During bootstrap, the curve is the primary path to SATO issuance and redemption.

As secondary liquidity develops, normal AMM pools may become the main trading venue. In that regime, the curve remains the canonical issuer when minting is profitable and a buyer of last resort when secondary bids are weak.

A trade only mints or burns when it routes through the curve pool. Trades through secondary pools do not change total supply or the curve reserve.

## Security Model

SATO minimizes discretionary trust assumptions.

The deployed contracts are designed without:

- owner-controlled monetary policy
- upgradeable proxy logic
- pause controls
- blacklist or whitelist controls
- transfer taxes
- treasury-directed issuance
- admin reserve withdrawal

This document is not an audit. It summarizes the verified protocol architecture and should be read together with the deployed source code.
