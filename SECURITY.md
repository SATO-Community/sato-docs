# Security Policy

This document summarizes the security posture, trust model, and responsible disclosure process for the SATO documentation repository and the deployed SATO protocol contracts.

SATO is designed around a simple principle:

> Security is achieved by minimizing trust assumptions rather than increasing administrative control.

The protocol favors deterministic smart contract execution, public verification, and transparent reserve accounting over discretionary administrative control.

## Scope

This security policy applies to the public SATO documentation and the following deployed protocol components:

| Component | Address / Role |
|---|---|
| SATO ERC-20 | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| SatoHook | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` |
| SatoSwapRouter | `0x06A645079cd4F3Bb38FfaD47f92180B8041145E3` |
| Uniswap v4 PoolManager | `0x000000000004444c5dc75cB358380D2e3dE08A90` |

The SATO ERC-20 token, `SatoHook`, and `SatoSwapRouter` should be reviewed together. Reviewing the token contract alone is not sufficient to understand the full protocol security model.

## Security Philosophy

SATO intentionally minimizes the amount of trust required to interact with the protocol.

Core security objectives include:

- minimize trust assumptions;
- eliminate discretionary monetary policy;
- reduce privileged administrative control;
- preserve deterministic protocol execution;
- keep reserve accounting transparent;
- maintain publicly auditable monetary behavior.

Rather than securing privileged operators, the protocol reduces or removes privileged roles wherever possible.

## Trust Model

The protocol is designed to reduce dependence on trusted parties.

Primary trust assumptions include:

- Ethereum Mainnet consensus;
- correct execution of deployed smart contracts;
- correct behavior of the Uniswap v4 PoolManager;
- public verification of deployed bytecode and source code;
- user selection of the correct token, hook, router, and market route.

No trusted operator is expected to manage SATO monetary policy.

## Administrative Model

SATO is designed without common ERC-20 administrative controls such as:

- upgradeable token proxy;
- administrative pause mechanism;
- blacklist;
- whitelist;
- ERC-20 transfer tax;
- treasury-controlled issuance;
- owner-controlled arbitrary burn.

The ERC-20 token separates token accounting from monetary policy. Minting and burning are restricted to the protocol minter, which is the deployed `SatoHook` contract.

## Protocol Authorization

SATO issuance and redemption are executed through protocol logic rather than discretionary administrator actions.

| Operation | Authorization Model |
|---|---|
| Mint | Executed through `SatoHook` during curve buys |
| Burn | Executed through `SatoHook` during curve sells |
| Reserve custody | ETH held by `SatoHook` |
| Direct curve routing | Assisted by `SatoSwapRouter` |
| Curve pricing | Defined by bonding-curve logic |
| Secondary market trading | External market behavior, separate from curve issuance |

`SatoSwapRouter` is an execution router. It does not control issuance, reserve custody, or curve pricing.

## Security Properties

The protocol is designed around the following security properties:

- verified deployed source code;
- narrow mint and burn authorization;
- no token-level blacklist or pause mechanism;
- no token-level transfer tax;
- transparent on-chain reserve custody;
- deterministic curve minting and burning;
- separation between token accounting and monetary policy;
- public reviewability of token, hook, router, and reserve state.

These properties reduce common owner-control risks, but they do not eliminate all technical, economic, or market risks.

## Threat Model

SATO is designed to reduce the impact of common administrative risks.

| Threat | Mitigation |
|---|---|
| Unauthorized monetary policy | Curve execution through `SatoHook` |
| Discretionary issuance | Minting restricted to the locked protocol minter |
| Administrative transfer restriction | No ERC-20 blacklist or pause logic |
| Hidden token tax | No ERC-20 transfer tax logic |
| Off-chain reserve management | Reserve held on-chain by `SatoHook` |
| Router custody confusion | Router is execution-only |
| Scanner misclassification | Documented false-positive and scanner-compatibility analysis |

Remaining risks include smart contract implementation risk, custom hook logic risk, Uniswap v4 settlement dependency, market liquidity risk, slippage, MEV, secondary market behavior, and the absence of an emergency pause or upgrade mechanism.

## Automated Security Analysis

Automated security scanners evaluate smart contracts using generalized heuristics.

SATO may trigger scanner warnings because its architecture includes:

- protocol-authorized minting;
- protocol-authorized burning;
- variable supply;
- ETH held by the hook contract;
- non-standard curve liquidity;
- router-based execution.

These signals should be reviewed in context. They are not automatically evidence of owner-controlled supply manipulation or malicious transfer restrictions.

Scanner output should be interpreted together with verified source code and the protocol documentation.

## Security References

Relevant documentation includes:

- `docs/protocol/01_Protocol_Overview.md`
- `docs/protocol/02_Architecture.md`
- `docs/protocol/03_ERC20.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/research/02_Protocol_Invariants.md`
- `docs/research/04_Economic_Security.md`
- `docs/research/05_Formal_Reasoning.md`
- `docs/security/01_Overview.md`
- `docs/security/02_Threat_Model.md`
- `docs/security/03_Trust_Model.md`
- `docs/security/04_Security_Guarantees.md`
- `docs/security/05_False_Positive_Analysis.md`
- `docs/security/06_Scanner_Compatibility.md`

## Responsible Disclosure

If you believe you have identified a legitimate security issue affecting the SATO protocol or this documentation, please use GitHub's private vulnerability reporting or Security Advisory process when available.

Please include:

- a clear description of the issue;
- affected contract, document, or component;
- steps to reproduce;
- expected behavior;
- observed behavior;
- potential impact;
- suggested mitigation, if available.

Please avoid public disclosure of unresolved security issues before the community has had a reasonable opportunity to review them.

## Limitations

This repository provides documentation and security analysis, but it is not a formal audit.

Users and reviewers should independently verify deployed contracts, source code, on-chain state, transaction behavior, market liquidity, and scanner output before relying on any security conclusion.
