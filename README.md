<p align="center">
  <img src="https://raw.githubusercontent.com/SATO-Community/sato-assets/main/sato_logo_cmc_200x200.png" width="140" alt="SATO Logo">
</p>

<h1 align="center">SATO</h1>

<p align="center">
<strong>Operator-Free Monetary Protocol on Ethereum</strong>
</p>

<p align="center">
Community-maintained technical documentation for the SATO protocol.
</p>

---

## Overview

SATO is an Ethereum-native monetary protocol that replaces discretionary human monetary control with deterministic smart contract execution.

Unlike conventional ERC-20 projects, SATO does not rely on treasury-controlled issuance, privileged administrators, governance-managed supply changes, or upgradeable token contracts.

SATO is built around a simple principle:

> Monetary policy should be enforced by code rather than by people.

The protocol uses a bonding-curve issuance and redemption model. Users can buy into the curve by depositing ETH and can sell back into the curve by burning SATO for ETH according to deterministic protocol rules.

---

## Live Protocol State

SATO is an Ethereum mainnet ERC-20 monetary experiment built around Uniswap v4 hook execution, bonding-curve mint/burn mechanics, and an ETH reserve.

Public metrics change over time, but the current protocol profile can be summarized as follows:

| Property | Value |
|---|---|
| Network | Ethereum Mainnet |
| Token | SATO |
| Curve Supply Asymptote | 21,000,000 SATO; this is not an ordinary fixed cap |
| Net ETH Reserve | Dynamic; see the live dashboard |
| Holders | Dynamic; see the live dashboard |
| Core Execution Layer | Uniswap v4 Hook |
| Reserve Holder | SatoHook |
| Operator Model | No active operator, no team treasury, no VC allocation |

SATO combines a visible ETH reserve with curve-based issuance and burn mechanics. The protocol is designed to run through verified smart contracts rather than discretionary team control.

### Live verification

The [SATO Live Dashboard](https://sato-live-dashboard.vercel.app/) presents current public metrics such as reserve, supply, holders, market price, mint and burn values, and Ethereum block height. Values shown by the dashboard are dynamic observations, not protocol constants. Use [Live State Verification](docs/data/01_Live_State_Verification.md) to check each metric against its primary source.

---

## Core Principles

### Operator-Free

SATO monetary policy is governed by deployed protocol logic rather than discretionary human decisions.

### Immutable

Core protocol behavior is designed to operate without proxy upgrades or administrative replacement of monetary logic.

### Reserve-Backed

Curve minting deposits ETH into the protocol reserve.

Curve redemption releases ETH from the protocol reserve according to deterministic bonding-curve rules.

### Transparent

Protocol behavior is executed on Ethereum and can be independently verified through deployed contracts, on-chain state, and transaction history.

---

## Network

| Property | Value |
|---|---|
| Blockchain | Ethereum Mainnet |
| Token Standard | ERC-20 |
| Primary Issuance Model | Bonding curve |
| Settlement Layer | Uniswap v4 PoolManager |

---

## Uniswap v4 Hook-Native Design

SATO is one of the early Ethereum mainnet assets built around Uniswap v4 hook execution.

Uniswap v4 hooks allow custom logic to run during the swap lifecycle. In SATO’s case, this mechanism is used to connect trading activity with bonding-curve issuance, burn/redemption behavior, and ETH reserve accounting.

This makes SATO different from a standard ERC-20 token. The core monetary logic is not based on manual emissions, team-controlled treasury actions, or discretionary administrator decisions. Instead, the protocol routes monetary behavior through the SatoHook contract and Uniswap v4 infrastructure.

---

## Deployed Contracts

| Component | Address / Link |
|---|---|
| SATO ERC-20 | [`0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09`](https://etherscan.io/token/0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09) |
| SatoHook | [`0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888`](https://etherscan.io/address/0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888) |
| SatoSwapRouter | [`0x06A645079cd4F3Bb38FfaD47f92180B8041145E3`](https://etherscan.io/address/0x06A645079cd4F3Bb38FfaD47f92180B8041145E3) |
| Uniswap v4 PoolManager | [`0x000000000004444c5dc75cB358380D2e3dE08A90`](https://etherscan.io/address/0x000000000004444c5dc75cB358380D2e3dE08A90) |

The SATO ERC-20 token, `SatoHook`, and `SatoSwapRouter` should be reviewed together. The ERC-20 token alone does not contain the full monetary system.

---

## Protocol Architecture

```text
                  Users
                    |
                    v
        SatoSwapRouter / Compatible Router
                    |
                    v
          Uniswap v4 PoolManager
                    |
                    v
                SatoHook
                    |
        +-----------+------------+
        |                        |
        v                        v
  Bonding Curve            Reserve Accounting
        |                        |
        +-----------+------------+
                    |
                    v
              SATO ERC-20
              Mint / Burn
```

The protocol separates token accounting from monetary policy.

- `SATO ERC-20` manages balances, transfers, total supply, and protocol-authorized minting and burning.
- `SatoHook` implements curve minting, curve redemption, reserve accounting, fee accounting, and protocol constraints.
- `SatoSwapRouter` is a verified execution router for direct curve buy and sell calls.
- Uniswap v4 `PoolManager` provides the settlement layer used by the hook execution path.

`SatoSwapRouter` does not control issuance, reserve custody, or curve pricing. Monetary enforcement remains inside `SatoHook`.

---

## Key Characteristics

- Operator-free monetary policy
- Ethereum Mainnet deployment
- ERC-20 token interface
- Uniswap v4 hook-native execution
- Bonding-curve issuance and redemption
- ETH reserve custody through `SatoHook`
- Verified execution router through `SatoSwapRouter`
- Publicly auditable smart contract behavior
- No token-level blacklist
- No token-level pause function
- No token-level transfer tax
- No upgradeable token proxy
- No discretionary owner-controlled minting after minter lock

---

## Documentation

This repository serves as the community-maintained technical reference for the SATO protocol.

### Listing

- [Token Information](docs/listing/01_Token_Information.md)

### Protocol

- [Protocol Overview](docs/protocol/01_Protocol_Overview.md)
- [Architecture](docs/protocol/02_Architecture.md)
- [ERC-20 Token](docs/protocol/03_ERC20.md)
- [SatoHook](docs/protocol/04_SatoHook.md)
- [Bonding Curve](docs/protocol/05_Bonding_Curve.md)
- [Reserve Model](docs/protocol/06_Reserve_Model.md)

### Developers

- [Forking and Independent Implementations](docs/developers/01_Forking_and_Independent_Implementations.md)
- [Contributing](CONTRIBUTING.md)

### Data

- [Live State Verification](docs/data/01_Live_State_Verification.md)

### Research

- [Monetary Model](docs/research/01_Monetary_Model.md)
- [Protocol Invariants](docs/research/02_Protocol_Invariants.md)
- [Game Theory](docs/research/03_Game_Theory.md)
- [Economic Security](docs/research/04_Economic_Security.md)
- [Formal Reasoning](docs/research/05_Formal_Reasoning.md)
- [References](docs/research/06_References.md)

### Security

- [Security Overview](docs/security/01_Overview.md)
- [Threat Model](docs/security/02_Threat_Model.md)
- [Trust Model](docs/security/03_Trust_Model.md)
- [Security Guarantees](docs/security/04_Security_Guarantees.md)
- [False Positive Analysis](docs/security/05_False_Positive_Analysis.md)
- [Scanner Compatibility](docs/security/06_Scanner_Compatibility.md)
- [Public Review Checklist](docs/security/07_Public_Review_Checklist.md)

---

## Developers and Independent Implementations

SATO's deployed token, hook, and router source packages are publicly readable and exact-match verified on Etherscan. Builders may inspect the architecture, reproduce its observable behavior, and develop independent implementations.

Public verification is not the same as an open-source license. Etherscan currently reports the deployed SATO contracts' license as `-NA-`, so this community documentation does not grant permission to copy or redistribute the original contract source. See [Forking and Independent Implementations](docs/developers/01_Forking_and_Independent_Implementations.md) for the technical workflow, attribution guidance, Uniswap v4 deployment constraints, and license boundary.

The hosted community dashboard is a separate interface. Its implementation and deployment files are not part of this public documentation repository.

---

## Suggested Reading Order

For a complete technical understanding of the protocol, start with:

1. [Protocol Overview](docs/protocol/01_Protocol_Overview.md)
2. [Architecture](docs/protocol/02_Architecture.md)
3. [ERC-20 Token](docs/protocol/03_ERC20.md)
4. [SatoHook](docs/protocol/04_SatoHook.md)
5. [Bonding Curve](docs/protocol/05_Bonding_Curve.md)
6. [Reserve Model](docs/protocol/06_Reserve_Model.md)

For public review, listing, wallet, scanner, or explorer checks, use:

- [Token Information](docs/listing/01_Token_Information.md)
- [Public Review Checklist](docs/security/07_Public_Review_Checklist.md)

Then review the research and security sections for economic analysis, protocol invariants, scanner interpretation, and threat modeling.

---

## Security

SATO minimizes trusted assumptions by replacing discretionary administrative control with deterministic protocol execution.

The protocol is designed around the following security properties:

- narrow mint and burn authorization;
- verified deployed source code;
- transparent on-chain reserve custody;
- deterministic curve execution;
- no token-level blacklist or pause mechanism;
- no token-level transfer tax;
- no upgradeable token proxy;
- separation between token accounting and monetary policy.

These properties reduce common owner-control risks, but they do not eliminate all technical, economic, or market risks.

This repository is not a formal audit. Users and reviewers should independently verify deployed contracts, source code, on-chain state, market liquidity, transaction behavior, and scanner output.

For the repository security policy, see [SECURITY.md](SECURITY.md).

---

## Scanner Interpretation

SATO may trigger automated scanner warnings because its architecture includes protocol-authorized minting, protocol-authorized burning, variable supply, ETH reserve custody, non-standard liquidity structure, and router-based execution.

These signals should be interpreted together with the verified ERC-20, hook, router, and reserve model.

Scanner output should be treated as a review signal, not as a final security conclusion.

---

## Official Resources

| Resource | Link |
|---|---|
| Website | https://sat0.org |
| Community Website | https://sat0.club |
| Whitepaper | https://sat0.org/whitepaper |
| Documentation | [docs/](docs/README.md) |
| Live State Verification | [docs/data/01_Live_State_Verification.md](docs/data/01_Live_State_Verification.md) |
| Developer Guide | [Forking and Independent Implementations](docs/developers/01_Forking_and_Independent_Implementations.md) |
| Contributing | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Token Information | [docs/listing/01_Token_Information.md](docs/listing/01_Token_Information.md) |
| Public Review Checklist | [docs/security/07_Public_Review_Checklist.md](docs/security/07_Public_Review_Checklist.md) |
| Security Policy | [SECURITY.md](SECURITY.md) |
| Assets Repository | https://github.com/SATO-Community/sato-assets |
| CoinGecko | https://www.coingecko.com/en/coins/sato-2 |
| CoinMarketCap DexScan | https://dex.coinmarketcap.com/token/ethereum/0x829f4b62eebe12af653b4dd4ffc480966f7d7f09/ |
| DeFiLlama | https://defillama.com/protocol/sato |
| DeFiLlama Hook-Based AMM | https://defillama.com/protocols/hook-based-amm |

Community channels support communication and discovery but do not control the protocol. Users should verify claims through the official website, deployed contract addresses, exact-match verified source packages, and on-chain state.

---

## Repository Structure

```text
README.md
CONTRIBUTING.md
SECURITY.md
LICENSE

docs/
|-- README.md
|-- data/
|-- developers/
|-- listing/
|-- protocol/
|-- research/
`-- security/
```

---

## License

This documentation repository is distributed under the MIT License. That license covers the community-authored material in this repository; it does not retroactively license the separately deployed SATO contract source packages, whose Etherscan license field is currently `-NA-`.

---

> This repository is maintained by community contributors and serves as a technical reference for the SATO protocol.
<p align="center">
  <img src="https://raw.githubusercontent.com/SATO-Community/sato-assets/main/sato_logo_cmc_200x200.png" width="140" alt="SATO Logo">
</p>

<h1 align="center">SATO</h1>

<p align="center">
<strong>Operator-Free Monetary Protocol on Ethereum</strong>
</p>

<p align="center">
Community-maintained technical documentation for the SATO protocol.
</p>

---

## Overview

SATO is an Ethereum-native monetary protocol that replaces discretionary human monetary control with deterministic smart contract execution.

Unlike conventional ERC-20 projects, SATO does not rely on treasury-controlled issuance, privileged administrators, governance-managed supply changes, or upgradeable token contracts.

SATO is built around a simple principle:

> Monetary policy should be enforced by code rather than by people.

The protocol uses a bonding-curve issuance and redemption model. Users can buy into the curve by depositing ETH and can sell back into the curve by burning SATO for ETH according to deterministic protocol rules.

---

## Current Public Snapshot

SATO is an Ethereum mainnet ERC-20 monetary experiment built around Uniswap v4 hook execution, bonding-curve mint/burn mechanics, and an ETH reserve.

Public metrics change over time, but the current protocol profile can be summarized as follows:

| Property | Value |
|---|---|
| Network | Ethereum Mainnet |
| Token | SATO |
| Max Supply | 21,000,000 SATO |
| Net ETH Reserve | Dynamic; see the live dashboard |
| Holders | Dynamic; see the live dashboard |
| Core Execution Layer | Uniswap v4 Hook |
| Reserve Holder | SatoHook |
| Operator Model | No active operator, no team treasury, no VC allocation |

SATO combines a visible ETH reserve with curve-based issuance and burn mechanics. The protocol is designed to run through verified smart contracts rather than discretionary team control.

### Live data reference

The [SATO Live Dashboard](https://sato-live-dashboard.vercel.app/) tracks the latest reserve, supply, holders, scarcity, market price, mint threshold, burn value, and Ethereum block. An observed snapshot on 16 July 2026 showed approximately 1,151.92 ETH reserve, 15.679M SATO supply, 8,939 holders, and 11.8% remaining reachable scarcity. These values change continuously and are not protocol constants.

---

## Core Principles

### Operator-Free

SATO monetary policy is governed by deployed protocol logic rather than discretionary human decisions.

### Immutable

Core protocol behavior is designed to operate without proxy upgrades or administrative replacement of monetary logic.

### Reserve-Backed

Curve minting deposits ETH into the protocol reserve.

Curve redemption releases ETH from the protocol reserve according to deterministic bonding-curve rules.

### Transparent

Protocol behavior is executed on Ethereum and can be independently verified through deployed contracts, on-chain state, and transaction history.

---

## Network

| Property | Value |
|---|---|
| Blockchain | Ethereum Mainnet |
| Token Standard | ERC-20 |
| Primary Issuance Model | Bonding curve |
| Settlement Layer | Uniswap v4 PoolManager |

---

## Uniswap v4 Hook-Native Design

SATO is one of the early Ethereum mainnet assets built around Uniswap v4 hook execution.

Uniswap v4 hooks allow custom logic to run during the swap lifecycle. In SATO’s case, this mechanism is used to connect trading activity with bonding-curve issuance, burn/redemption behavior, and ETH reserve accounting.

This makes SATO different from a standard ERC-20 token. The core monetary logic is not based on manual emissions, team-controlled treasury actions, or discretionary administrator decisions. Instead, the protocol routes monetary behavior through the SatoHook contract and Uniswap v4 infrastructure.

---

## Deployed Contracts

| Component | Address / Link |
|---|---|
| SATO ERC-20 | [`0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09`](https://etherscan.io/token/0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09) |
| SatoHook | [`0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888`](https://etherscan.io/address/0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888) |
| SatoSwapRouter | [`0x06A645079cd4F3Bb38FfaD47f92180B8041145E3`](https://etherscan.io/address/0x06A645079cd4F3Bb38FfaD47f92180B8041145E3) |
| Uniswap v4 PoolManager | [`0x000000000004444c5dc75cB358380D2e3dE08A90`](https://etherscan.io/address/0x000000000004444c5dc75cB358380D2e3dE08A90) |

The SATO ERC-20 token, `SatoHook`, and `SatoSwapRouter` should be reviewed together. The ERC-20 token alone does not contain the full monetary system.

---

## Protocol Architecture

```text
                  Users
                    |
                    v
        SatoSwapRouter / Compatible Router
                    |
                    v
          Uniswap v4 PoolManager
                    |
                    v
                SatoHook
                    |
        +-----------+------------+
        |                        |
        v                        v
  Bonding Curve            Reserve Accounting
        |                        |
        +-----------+------------+
                    |
                    v
              SATO ERC-20
              Mint / Burn
```

The protocol separates token accounting from monetary policy.

- `SATO ERC-20` manages balances, transfers, total supply, and protocol-authorized minting and burning.
- `SatoHook` implements curve minting, curve redemption, reserve accounting, fee accounting, and protocol constraints.
- `SatoSwapRouter` is a verified execution router for direct curve buy and sell calls.
- Uniswap v4 `PoolManager` provides the settlement layer used by the hook execution path.

`SatoSwapRouter` does not control issuance, reserve custody, or curve pricing. Monetary enforcement remains inside `SatoHook`.

---

## Key Characteristics

- Operator-free monetary policy
- Ethereum Mainnet deployment
- ERC-20 token interface
- Uniswap v4 hook-native execution
- Bonding-curve issuance and redemption
- ETH reserve custody through `SatoHook`
- Verified execution router through `SatoSwapRouter`
- Publicly auditable smart contract behavior
- No token-level blacklist
- No token-level pause function
- No token-level transfer tax
- No upgradeable token proxy
- No discretionary owner-controlled minting after minter lock

---

## Documentation

This repository serves as the community-maintained technical reference for the SATO protocol.

### Listing

- [Token Information](docs/listing/01_Token_Information.md)

### Protocol

- [Protocol Overview](docs/protocol/01_Protocol_Overview.md)
- [Architecture](docs/protocol/02_Architecture.md)
- [ERC-20 Token](docs/protocol/03_ERC20.md)
- [SatoHook](docs/protocol/04_SatoHook.md)
- [Bonding Curve](docs/protocol/05_Bonding_Curve.md)
- [Reserve Model](docs/protocol/06_Reserve_Model.md)

### Research

- [Monetary Model](docs/research/01_Monetary_Model.md)
- [Protocol Invariants](docs/research/02_Protocol_Invariants.md)
- [Game Theory](docs/research/03_Game_Theory.md)
- [Economic Security](docs/research/04_Economic_Security.md)
- [Formal Reasoning](docs/research/05_Formal_Reasoning.md)
- [References](docs/research/06_References.md)

### Security

- [Security Overview](docs/security/01_Overview.md)
- [Threat Model](docs/security/02_Threat_Model.md)
- [Trust Model](docs/security/03_Trust_Model.md)
- [Security Guarantees](docs/security/04_Security_Guarantees.md)
- [False Positive Analysis](docs/security/05_False_Positive_Analysis.md)
- [Scanner Compatibility](docs/security/06_Scanner_Compatibility.md)
- [Public Review Checklist](docs/security/07_Public_Review_Checklist.md)

---

## Suggested Reading Order

For a complete technical understanding of the protocol, start with:

1. [Protocol Overview](docs/protocol/01_Protocol_Overview.md)
2. [Architecture](docs/protocol/02_Architecture.md)
3. [ERC-20 Token](docs/protocol/03_ERC20.md)
4. [SatoHook](docs/protocol/04_SatoHook.md)
5. [Bonding Curve](docs/protocol/05_Bonding_Curve.md)
6. [Reserve Model](docs/protocol/06_Reserve_Model.md)

For public review, listing, wallet, scanner, or explorer checks, use:

- [Token Information](docs/listing/01_Token_Information.md)
- [Public Review Checklist](docs/security/07_Public_Review_Checklist.md)

Then review the research and security sections for economic analysis, protocol invariants, scanner interpretation, and threat modeling.

---

## Security

SATO minimizes trusted assumptions by replacing discretionary administrative control with deterministic protocol execution.

The protocol is designed around the following security properties:

- narrow mint and burn authorization;
- verified deployed source code;
- transparent on-chain reserve custody;
- deterministic curve execution;
- no token-level blacklist or pause mechanism;
- no token-level transfer tax;
- no upgradeable token proxy;
- separation between token accounting and monetary policy.

These properties reduce common owner-control risks, but they do not eliminate all technical, economic, or market risks.

This repository is not a formal audit. Users and reviewers should independently verify deployed contracts, source code, on-chain state, market liquidity, transaction behavior, and scanner output.

For the repository security policy, see [SECURITY.md](SECURITY.md).

---

## Scanner Interpretation

SATO may trigger automated scanner warnings because its architecture includes protocol-authorized minting, protocol-authorized burning, variable supply, ETH reserve custody, non-standard liquidity structure, and router-based execution.

These signals should be interpreted together with the verified ERC-20, hook, router, and reserve model.

Scanner output should be treated as a review signal, not as a final security conclusion.

---

## Official Resources

| Resource | Link |
|---|---|
| Website | https://sat0.org |
| Community Website | https://sat0.club |
| Whitepaper | https://sat0.org/whitepaper |
| Documentation | [docs/](docs/README.md) |
| Token Information | [docs/listing/01_Token_Information.md](docs/listing/01_Token_Information.md) |
| Public Review Checklist | [docs/security/07_Public_Review_Checklist.md](docs/security/07_Public_Review_Checklist.md) |
| Security Policy | [SECURITY.md](SECURITY.md) |
| Assets Repository | https://github.com/SATO-Community/sato-assets |
| CoinGecko | https://www.coingecko.com/en/coins/sato-2 |
| CoinMarketCap DexScan | https://dex.coinmarketcap.com/token/ethereum/0x829f4b62eebe12af653b4dd4ffc480966f7d7f09/ |
| DeFiLlama | https://defillama.com/protocol/sato |
| DeFiLlama Hook-Based AMM | https://defillama.com/protocols/hook-based-amm |

SATO does not currently rely on official social channels for protocol verification. Users should verify information through the official website, deployed contract addresses, and public source code.

---

## Repository Structure

```text
README.md
SECURITY.md
LICENSE

docs/
|-- README.md
|-- listing/
|-- protocol/
|-- research/
`-- security/
```

---

## License

This repository is distributed under the MIT License.

---

> This repository is maintained by community contributors and serves as a technical reference for the SATO protocol.
