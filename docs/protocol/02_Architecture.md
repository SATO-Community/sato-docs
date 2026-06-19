# Architecture

SATO is structured as a separation between token accounting and monetary execution.

The ERC-20 contract records balances and supply. The hook contract defines how supply can be created or destroyed. The curve library provides the deterministic math used by the hook.

## Components

| Component | Address / Source | Responsibility |
| --- | --- | --- |
| `SatoToken` | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` | ERC-20 balances, transfers, total supply, protocol-authorized mint and burn |
| `SatoHook` | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` | Uniswap v4 hook, curve execution, ETH reserve custody |
| `Curve` | internal library | Forward minting curve and inverse redemption math |
| Uniswap v4 `PoolManager` | `0x000000000004444c5dc75cB358380D2e3dE08A90` | Settlement layer for hook-routed swaps |

## System Diagram

```text
                 User / Router
                      |
                      v
              Uniswap v4 PoolManager
                      |
                      v
                  SatoHook
             +--------+--------+
             |                 |
             v                 v
          Curve             SatoToken
     pricing math       ERC-20 accounting
             |                 |
             +--------+--------+
                      |
                      v
        Mint / Burn and ETH Reserve Updates
```

## Token Accounting Layer

`SatoToken` is a minimal ERC-20 accounting contract.

It is responsible for balances, transfers, allowances, total supply, mint records, and burn records.

It does not implement owner-controlled monetary policy, pause controls, blacklist controls, transfer taxes, proxy upgrades, or treasury issuance.

The contract has one designated `minter`. The minter can be set once and cannot be changed afterward. In the deployed protocol, the intended minter is `SatoHook`.

## Monetary Execution Layer

`SatoHook` is the protocol execution layer.

It is a Uniswap v4 hook that handles ETH -> SATO buys and SATO -> ETH sells. It uses `BeforeSwapDelta` to take over swap accounting instead of relying on ordinary AMM liquidity.

The hook validates the ETH/SATO pool, rejects unsupported pool configurations, prevents external liquidity additions, executes curve-based buys and sells, mints SATO through the locked minter path, burns SATO during curve redemption, tracks cumulative curve state, and holds ETH reserves used for redemptions.

## Hook Permissions

The deployed hook declares only the permissions needed for the protocol:

| Hook permission | Status |
| --- | --- |
| `beforeInitialize` | enabled |
| `beforeAddLiquidity` | enabled |
| `beforeSwap` | enabled |
| `beforeSwapReturnDelta` | enabled |
| all other hook permissions | disabled |

`beforeAddLiquidity` always reverts. This means external liquidity cannot be added to the curve pool.

## Pool Initialization

The hook accepts only the configured ETH/SATO pool.

During initialization, the hook validates that `currency0` is native ETH, `currency1` is SATO, the pool fee is `3000` (`0.3%`), and the hook address is the deployed `SatoHook`.

If any condition fails, initialization reverts.

## Buy Flow

A buy is an exact-input ETH -> SATO swap through the curve pool.

```text
User provides ETH
      |
      v
Router pre-settles ETH to PoolManager
      |
      v
SatoHook receives beforeSwap
      |
      v
Fee is separated
      |
      v
Curve calculates fair SATO output
      |
      v
SatoHook mints SATO to PoolManager
      |
      v
PoolManager routes SATO to buyer
      |
      v
ETH reserve and curve state update
```

Single buys are capped at `5 ETH`. Exact-output swaps are not supported.

## Sell Flow

A sell is an exact-input SATO -> ETH swap through the curve pool.

```text
User provides SATO
      |
      v
Router pre-settles SATO to PoolManager
      |
      v
SatoHook receives beforeSwap
      |
      v
Hook calculates fair curve redemption
      |
      v
SatoHook takes and burns SATO
      |
      v
SatoHook settles ETH to PoolManager
      |
      v
PoolManager routes ETH to seller
      |
      v
Reserve and curve state update
```

A wallet cannot sell in the same block as its last buy through the curve.

## Reserve Architecture

ETH received from buys is held by the hook contract.

The sell-side curve reserve is calculated as:

```text
curveReserveEth = hook ETH balance - feesAccrued
```

The reserve is not managed by an administrator. It moves out through curve redemptions when SATO is burned.

## Secondary Markets

The curve pool is not the same as secondary AMM markets.

A swap through the curve pool may mint or burn SATO. A swap through a secondary pool only trades existing SATO against AMM liquidity and does not change total supply or the curve reserve.

## Design Summary

The architecture is designed to keep each layer narrow:

- `SatoToken` records token state.
- `SatoHook` enforces monetary execution.
- `Curve` computes deterministic pricing.
- Uniswap v4 `PoolManager` provides settlement rails.

This structure removes discretionary monetary control from token holders, operators, or administrators and places issuance and redemption behavior in verified contract logic.
