# SATO Documentation

Technical documentation for the SATO protocol.

This documentation covers the protocol architecture, bonding-curve monetary design, reserve model, research notes, security interpretation, and listing information for scanners, wallets, explorers, and token information platforms.

## Core Contracts

| Component | Address / Role |
|---|---|
| SATO ERC-20 | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| SatoHook | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` |
| SatoSwapRouter | `0x06A645079cd4F3Bb38FfaD47f92180B8041145E3` |
| Uniswap v4 PoolManager | `0x000000000004444c5dc75cB358380D2e3dE08A90` |

## Sections

### Listing

- [Token Information](listing/01_Token_Information.md)

### Protocol

- [Protocol Overview](protocol/01_Protocol_Overview.md)
- [Architecture](protocol/02_Architecture.md)
- [ERC-20 Token](protocol/03_ERC20.md)
- [SatoHook](protocol/04_SatoHook.md)
- [Bonding Curve](protocol/05_Bonding_Curve.md)
- [Reserve Model](protocol/06_Reserve_Model.md)

### Research

- [Monetary Model](research/01_Monetary_Model.md)
- [Protocol Invariants](research/02_Protocol_Invariants.md)
- [Game Theory](research/03_Game_Theory.md)
- [Economic Security](research/04_Economic_Security.md)
- [Formal Reasoning](research/05_Formal_Reasoning.md)
- [References](research/06_References.md)

### Security

- [Security Overview](security/01_Overview.md)
- [Threat Model](security/02_Threat_Model.md)
- [Trust Model](security/03_Trust_Model.md)
- [Security Guarantees](security/04_Security_Guarantees.md)
- [False Positive Analysis](security/05_False_Positive_Analysis.md)
- [Scanner Compatibility](security/06_Scanner_Compatibility.md)
- [Public Review Checklist](security/07_Public_Review_Checklist.md)

## Suggested Reading Order

For a complete technical understanding of SATO, start with:

1. [Protocol Overview](protocol/01_Protocol_Overview.md)
2. [Architecture](protocol/02_Architecture.md)
3. [ERC-20 Token](protocol/03_ERC20.md)
4. [SatoHook](protocol/04_SatoHook.md)
5. [Bonding Curve](protocol/05_Bonding_Curve.md)
6. [Reserve Model](protocol/06_Reserve_Model.md)

For listing submissions, wallet metadata, scanner review, or explorer context, use:

- [Token Information](listing/01_Token_Information.md)
- [Public Review Checklist](security/07_Public_Review_Checklist.md)

After that, review the research and security sections for deeper economic, formal, and scanner-related analysis.

## Notes

SATO uses a bonding-curve issuance model rather than a standard fixed-supply ERC-20 launch.

Minting and burning are expected protocol behavior and are routed through the verified `SatoHook` contract. The verified `SatoSwapRouter` is an execution router for direct curve buy and sell calls; it does not control issuance, reserve custody, or curve pricing.

This documentation is not a security audit. Users and reviewers should independently verify deployed contracts, source code, on-chain state, market liquidity, and transaction behavior.
