# Live State Verification

## Purpose

SATO reserve, supply, holder, price, volume, and block values change continuously. This page explains where to verify current values instead of storing a snapshot that quickly becomes outdated.

## Primary Contract Sources

| Metric | Primary source | Meaning |
| --- | --- | --- |
| Curve reserve | `SatoHook.curveReserveEth()` | ETH reported as the curve reserve after excluding `feesAccrued` |
| Accrued fees | `SatoHook.feesAccrued()` | ETH tracked separately by the hook's fee accounting |
| Curve position | `SatoHook.ethCum()` | Cumulative ETH position used by the fair curve |
| Fair supply state | `SatoHook.totalMintedFair()` | Canonical supply position used by curve calculations |
| Actual token supply | `SatoToken.totalSupply()` | Current ERC-20 supply after mints and burns |
| Issuance status | `SatoHook.selfDeprecated()` | Whether new curve buys have been disabled at the protocol threshold |
| Raw hook balance | Ethereum address balance | All ETH held at the hook address; do not automatically label this as curve reserve |

All integer values returned in wei or token base units use 18 decimals unless the verified contract specifies otherwise:

```text
display value = raw integer / 10^18
```

## Contract Links

- [SatoHook - Read Contract](https://etherscan.io/address/0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888#readContract)
- [SatoHook - Verified Source](https://etherscan.io/address/0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888#code)
- [SATO Token](https://etherscan.io/token/0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09)
- [SatoSwapRouter](https://etherscan.io/address/0x06A645079cd4F3Bb38FfaD47f92180B8041145E3#code)

## Metrics That Are Not Contract Variables

Some public metrics are derived by indexers or markets rather than stored by SATO contracts:

| Metric | Suggested reference | Limitation |
| --- | --- | --- |
| Holder count | Etherscan token holders | Indexer-derived and may update with delay |
| Market price | CoinGecko or active DEX market | Varies by venue, liquidity, and routing |
| Trading volume | DEX indexer or verified on-chain query | Depends on pool coverage and methodology |
| USD reserve value | `curveReserveEth()` multiplied by a stated ETH/USD source | Changes with both reserve and ETH price |
| TVL | [DeFiLlama SATO](https://defillama.com/protocol/sato) | Third-party methodology; cross-check the hook function |

## Convenience Views

- [SATO Live Dashboard](https://sato-live-dashboard.vercel.app/)
- [DeFiLlama](https://defillama.com/protocol/sato)
- [CoinGecko](https://www.coingecko.com/en/coins/sato-2)

These interfaces improve readability but do not replace the deployed contract as the source of truth for protocol state.

## Publishing a Snapshot

When a dated snapshot is necessary, include all of the following:

```text
Metric definition:
Raw value:
Displayed value:
Ethereum block:
UTC timestamp:
Contract or indexer source:
Calculation notes:
```

Do not reuse a snapshot as though it were current. Readers should be directed back to this verification process.

## Important Distinctions

- `address(SatoHook).balance` is the raw ETH balance.
- `curveReserveEth()` is the reserve view used for protocol reporting.
- `totalMintedFair()` is the fair-curve accounting state.
- `SatoToken.totalSupply()` is the live ERC-20 supply.
- Holder count, market price, and volume are indexer or market observations.

Keeping these definitions separate prevents gross reserve, curve reserve, fair supply, and actual supply from being mixed together.
