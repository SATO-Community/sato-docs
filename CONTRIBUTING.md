# Contributing to SATO Documentation

Thank you for helping improve the community-maintained SATO technical reference.

## What Belongs Here

Contributions may include:

- corrections supported by deployed contract source or on-chain state
- clearer explanations of protocol architecture
- reserve, curve, and supply terminology
- security review notes that avoid exploit-ready disclosure
- listing, wallet, explorer, and scanner references
- developer guidance for independent implementations
- translations that preserve technical meaning

The hosted community dashboard source, deployment configuration, private infrastructure, credentials, and API internals are outside the scope of this public repository.

## Evidence Standard

Technical claims should link to a primary source whenever possible:

1. verified deployed contract source
2. Ethereum transaction, event, or read-only contract state
3. official Uniswap v4 documentation
4. SATO whitepaper or protocol documentation
5. clearly identified third-party analytics

Do not present a scanner score, market page, social post, or dashboard value as canonical contract state.

## Dynamic Data Policy

Reserve, supply, holder, price, volume, and block values change over time.

When a snapshot is useful, include:

- the exact metric definition
- Ethereum block number
- UTC timestamp
- data source
- units and decimal handling

Evergreen documents should normally explain how to verify a value rather than hardcode a number that will become stale.

## Terminology

Use these terms precisely:

- `curveReserveEth()`: curve reserve reported by the deployed hook
- `feesAccrued`: separately tracked fee accounting
- `totalMintedFair`: canonical fair-curve supply state
- `totalSupply()`: current ERC-20 supply
- `K_SUPPLY`: curve supply asymptote, not an ordinary fixed cap
- secondary market: a pool that does not update the SATO curve state

## Pull Request Checklist

Before opening a pull request:

- verify every address and link
- distinguish observation from interpretation
- remove unsupported promotional claims
- avoid stale live metrics unless timestamped
- confirm relative links render correctly
- explain why the change improves technical accuracy
- do not include secrets, wallet material, private dashboard code, or personal data

## Security Findings

Do not publish exploit-ready details for a potentially actionable vulnerability.

Follow [SECURITY.md](SECURITY.md) and use GitHub private vulnerability reporting when available. Public documentation can be updated after responsible disclosure and remediation context are established.

## Licensing

Community-authored documentation in this repository is covered by the repository's MIT License.

The deployed SATO contract source packages are publicly exact-match verified, but their Etherscan license field is currently `-NA-`. This repository cannot grant rights over code whose copyright is held elsewhere. See [Forking and Independent Implementations](docs/developers/01_Forking_and_Independent_Implementations.md).

