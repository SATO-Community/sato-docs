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

# Overview

SATO is an Ethereum-native monetary protocol that replaces discretionary human governance with immutable smart contract execution.

Unlike conventional ERC-20 projects, SATO does not rely on privileged administrators, treasury-controlled issuance, governance mechanisms, or upgradeable contracts. Instead, monetary issuance and redemption are governed entirely by immutable protocol logic.

The protocol implements deterministic issuance and redemption through immutable smart contracts, eliminating the need for treasury-controlled monetary policy or privileged administrative intervention.

SATO is built around a simple principle:

> **Monetary policy should be enforced by code rather than by people.**

Instead of relying on governance votes, multisignature wallets, foundation-controlled distribution, or discretionary monetary management, SATO defines protocol behavior directly in immutable smart contracts.

---

# Core Principles

### Operator-Free

Monetary issuance is governed by immutable protocol logic rather than discretionary human decisions.

### Immutable

Core protocol behavior is immutable after deployment. The protocol does not rely on proxy architecture or upgradeable contracts.

### Reserve-Backed

Every mint operation deposits ETH into the protocol reserve.

Every burn operation redeems ETH from the protocol reserve according to deterministic protocol rules.

### Transparent

All protocol operations are executed entirely on Ethereum.

Every transaction can be independently verified on-chain.

---

# Key Characteristics

* Operator-free monetary policy
* Immutable protocol contracts
* Deterministic bonding curve
* Reserve-backed issuance and redemption
* Publicly auditable monetary policy
* Fully transparent on-chain execution
* No privileged administrator

---

# Network

| Property       | Value            |
| -------------- | ---------------- |
| Blockchain     | Ethereum Mainnet |
| Token Standard | ERC-20           |

---

# Smart Contracts

## SATO ERC-20

**Contract**

https://etherscan.io/token/0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09

Responsible for:

* Token balances
* Transfers
* Total supply
* Protocol-authorized minting
* Protocol-authorized burning

> Detailed protocol architecture and the SatoHook contract are documented separately within this repository.

---

# Protocol Architecture

```text
                Users
                  │
                  ▼
             SatoHook
                  │
           Bonding Curve
                  │
          Reserve Accounting
                  │
             Mint / Burn
                  │
                  ▼
             SATO ERC-20
```

The protocol separates monetary policy from token accounting.

* **SatoHook** implements reserve accounting, bonding curve execution, issuance, and redemption.
* **SATO ERC-20** manages balances, transfers, and protocol-authorized minting and burning.

This separation allows each contract to maintain a single, well-defined responsibility while keeping monetary policy deterministic, transparent, and publicly auditable.

---

# Design Goals

The protocol is designed to minimize trust assumptions while maximizing transparency and deterministic execution.

Primary objectives include:

* Minimize trust assumptions
* Eliminate discretionary monetary policy
* Remove privileged administrative control
* Maintain transparent reserve accounting
* Enforce protocol behavior entirely on-chain
* Keep monetary issuance predictable and publicly auditable

---

# Documentation

This repository serves as the community-maintained technical reference for the SATO protocol.

Documentation includes:

* Protocol Overview
* Protocol Architecture
* ERC-20 Specification
* SatoHook Specification
* Bonding Curve
* Reserve Model
* Security Architecture
* Threat Model
* Trust Model
* Protocol Invariants
* False Positive Analysis
* Technical Research

Additional documentation will continue to expand alongside protocol research.

---

# Security Philosophy

SATO minimizes trusted assumptions by replacing discretionary administrative control with immutable protocol execution.

The protocol is designed around the following principles:

* No privileged administrator
* No upgradeable contracts
* No proxy architecture
* Deterministic monetary policy
* Transparent smart contract execution
* Publicly auditable reserve accounting

The protocol favors deterministic execution, transparency, and publicly auditable monetary rules over discretionary administrative control.

---

# Repository Structure

```text
README.md
SECURITY.md

docs/
├── protocol/
├── security/
└── research/

diagrams/

papers/
```

---

# Resources

| Resource   | Link                         |
| ---------- | ---------------------------- |
| Website    | https://sat0.org             |
| Whitepaper | https://sat0.org/whitepaper  |
| Community  | https://sat0.club            |
| Telegram   | https://t.me/sat0org         |
| X          | https://x.com/sat0_community |

---

# License

This repository is distributed under the MIT License.

---

> This repository is maintained by community contributors and serves as the primary technical reference for the SATO protocol.
