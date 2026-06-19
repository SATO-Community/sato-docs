# Token Information

This document provides a concise reference for SATO token listing submissions, wallet metadata, scanner context, explorer review, and public protocol verification.

## Basic Information

| Field | Value |
|---|---|
| Token Name | sato |
| Token Symbol | sato |
| Network | Ethereum Mainnet |
| Token Standard | ERC-20 |
| Decimals | 18 |
| Contract Address | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| Website | https://sat0.org |
| Whitepaper | https://sat0.org/whitepaper |
| Documentation | https://github.com/SATO-Community/sato-docs |
| Assets Repository | https://github.com/SATO-Community/sato-assets |
| Logo URL | https://raw.githubusercontent.com/SATO-Community/sato-assets/main/sato_logo_cmc_200x200.png |

## Token Contract

| Item | Value |
|---|---|
| Contract | SATO ERC-20 |
| Address | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| Etherscan | https://etherscan.io/token/0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09 |
| Role | Token balances, transfers, total supply, protocol-authorized minting and burning |

The SATO ERC-20 contract manages token accounting. It should be reviewed together with `SatoHook`, because the hook contract contains the primary monetary logic.

## Protocol Contracts

| Component | Address / Link | Role |
|---|---|---|
| SATO ERC-20 | [`0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09`](https://etherscan.io/token/0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09) | Token accounting |
| SatoHook | [`0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888`](https://etherscan.io/address/0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888) | Curve execution, minting, burning, reserve accounting |
| SatoSwapRouter | [`0x06A645079cd4F3Bb38FfaD47f92180B8041145E3`](https://etherscan.io/address/0x06A645079cd4F3Bb38FfaD47f92180B8041145E3) | Direct curve buy/sell execution router |
| Uniswap v4 PoolManager | [`0x000000000004444c5dc75cB358380D2e3dE08A90`](https://etherscan.io/address/0x000000000004444c5dc75cB358380D2e3dE08A90) | External settlement layer |

## Short Description

SATO is an Ethereum-native monetary protocol with bonding-curve issuance and redemption. Monetary behavior is enforced by deployed smart contracts rather than discretionary administrators.

## Extended Description

SATO is an operator-free monetary protocol on Ethereum.

The protocol uses a bonding-curve model where users can buy into the curve by depositing ETH and can sell back into the curve by burning SATO for ETH according to deterministic rules.

SATO separates token accounting from monetary policy:

- the ERC-20 token manages balances, transfers, and total supply;
- `SatoHook` manages curve issuance, redemption, reserve accounting, and protocol constraints;
- `SatoSwapRouter` provides a verified execution path for direct curve buys and sells;
- Uniswap v4 PoolManager provides the settlement layer used by the hook execution path.

The protocol is designed to minimize discretionary administrative control and make monetary behavior publicly verifiable on-chain.

## Logo Information

| Field | Value |
|---|---|
| Logo File | `sato_logo_cmc_200x200.png` |
| Format | PNG |
| Size | 200 x 200 |
| Raw URL | https://raw.githubusercontent.com/SATO-Community/sato-assets/main/sato_logo_cmc_200x200.png |

## Listing Notes

SATO may differ from standard fixed-supply ERC-20 projects because its supply changes through protocol-authorized curve minting and burning.

Listing platforms should review the ERC-20 token together with the verified `SatoHook` contract and the verified `SatoSwapRouter` contract.

Important notes:

- SATO has no premine model in the documented curve issuance design.
- Minting is part of curve-based issuance.
- Burning is part of curve-based redemption.
- The ETH curve reserve is held by `SatoHook`.
- `SatoSwapRouter` is an execution router, not the reserve holder.
- Scanner output should be interpreted with source-level protocol context.

## Security and Scanner Context

Automated scanners may flag SATO because the protocol includes:

- mint function;
- burn function;
- variable supply;
- protocol minter role;
- ETH held by the hook contract;
- non-standard curve liquidity;
- router-based execution.

These signals should be interpreted in context.

SATO’s mint and burn behavior is part of the bonding-curve protocol model. The ERC-20 token does not by itself define arbitrary owner-controlled monetary policy.

Scanner output should be treated as a review signal, not as a final security conclusion.

## Administrative Model

SATO is designed without common token-level administrative controls such as:

| Feature | Status |
|---|---|
| Token-level blacklist | Not part of the ERC-20 token model |
| Token-level pause | Not part of the ERC-20 token model |
| Token-level transfer tax | Not part of the ERC-20 token model |
| Upgradeable token proxy | Not part of the token contract model |
| Owner-controlled arbitrary minting | Not part of the documented post-minter-lock model |
| Owner-controlled arbitrary burning | Not part of the documented model |

This does not mean the protocol is risk-free. It means the main risks should be analyzed at the protocol, hook, curve, reserve, router, market, and execution layers rather than treated as standard token owner-control risks.

## Official Verification Sources

| Source | Link |
|---|---|
| Website | https://sat0.org |
| Whitepaper | https://sat0.org/whitepaper |
| Documentation | https://github.com/SATO-Community/sato-docs |
| Assets | https://github.com/SATO-Community/sato-assets |
| ERC-20 Contract | https://etherscan.io/token/0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09 |
| SatoHook Contract | https://etherscan.io/address/0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888 |
| SatoSwapRouter Contract | https://etherscan.io/address/0x06A645079cd4F3Bb38FfaD47f92180B8041145E3 |

## Social Channel Note

SATO does not currently rely on official social channels for protocol verification.

Users, wallets, scanners, and listing platforms should verify SATO through:

- the official website;
- the deployed token address;
- verified contract source code;
- public documentation;
- on-chain state.

Claims from unofficial accounts, chats, groups, or social profiles should be verified against deployed contracts and public source code.

## Suggested Listing Summary

SATO is an Ethereum-native bonding-curve monetary protocol. The SATO ERC-20 token is paired with a verified `SatoHook` contract that manages curve issuance, redemption, reserve accounting, and protocol constraints. Direct curve buys and sells may be executed through the verified `SatoSwapRouter`. The protocol is designed to minimize discretionary administrative control and make monetary behavior publicly auditable on-chain.

## Related Documentation

- `docs/README.md`
- `docs/protocol/01_Protocol_Overview.md`
- `docs/protocol/02_Architecture.md`
- `docs/protocol/03_ERC20.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/security/05_False_Positive_Analysis.md`
- `docs/security/06_Scanner_Compatibility.md`

## Limitations

This document is informational and is not a security audit.

Users, reviewers, wallets, scanners, and listing platforms should independently verify deployed contracts, source code, on-chain state, market liquidity, transaction behavior, and scanner output before relying on any conclusion.
