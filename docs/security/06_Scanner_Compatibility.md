# Scanner Compatibility

This document explains how SATO should be interpreted by automated token scanners, wallet risk engines, DEX interfaces, and public smart contract analysis tools.

SATO uses a bonding-curve issuance model with a verified ERC-20 token, a verified Uniswap v4 hook, and a verified execution router. Because this architecture differs from a standard fixed-supply ERC-20 launch, some scanner results may require manual interpretation.

## Scope

| Component | Address / Role |
|---|---|
| SATO ERC-20 | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| SatoHook | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` |
| SatoSwapRouter | `0x06A645079cd4F3Bb38FfaD47f92180B8041145E3` |
| Uniswap v4 PoolManager | External settlement layer |
| Curve Reserve | ETH held by `SatoHook`, excluding accrued fees |

## Compatibility Principle

Most automated scanners are optimized for standard ERC-20 tokens, fixed-supply launches, and conventional DEX liquidity pools.

SATO is different.

The token supply is created and reduced through the bonding curve. Minting and burning are expected protocol behavior, not automatically evidence of owner-controlled supply manipulation.

Scanner output should therefore be interpreted together with:

- verified ERC-20 source code;
- verified `SatoHook` source code;
- verified `SatoSwapRouter` source code;
- on-chain minter configuration;
- reserve accounting;
- Uniswap v4 pool and hook behavior.

## Expected Scanner Signals

| Scanner Signal | Expected? | SATO Interpretation |
|---|---:|---|
| Mint function exists | Yes | Required for curve-based issuance |
| Burn function exists | Yes | Required for curve-based redemption |
| Supply can change | Yes | Supply expands and contracts through the curve |
| Minter role exists | Yes | Minter is locked to `SatoHook` |
| Contract holds ETH | Yes | ETH reserve is held by `SatoHook` |
| No traditional LP lock | Possible | Primary liquidity is the curve reserve, not a standard LP lock |
| Router interaction | Yes | `SatoSwapRouter` routes execution only |
| Transfer tax detected | Should be reviewed | Curve fees are not ERC-20 transfer taxes |
| Honeypot warning | Should be reviewed | Curve rules are not owner-controlled sell blocks |
| Owner/admin warning | Should be reviewed | Check verified source and minter lock state |

## Platform Interpretation

### Etherscan

Etherscan should be used as the primary public verification source for deployed contract code.

Relevant checks:

- confirm the ERC-20 token source is verified;
- confirm the contract name is `SatoToken`;
- confirm the hook source is verified;
- confirm the contract name is `SatoHook`;
- confirm the router source is verified;
- confirm the contract name is `SatoSwapRouter`;
- confirm the ERC-20 minter is the deployed hook contract;
- confirm there is no proxy replacement pattern for the token contract.

Etherscan verification does not by itself prove economic safety, but it allows users and reviewers to inspect the deployed source code directly.

### Sourcify

Sourcify can be used as an additional source-verification reference where available.

Sourcify verification is useful because it helps confirm that deployed bytecode corresponds to published source code.

Reviewers should still verify the relevant deployed address and chain before relying on any source match.

### GoPlus Security

GoPlus Security and similar scanner platforms are useful for quick token and contract risk screening.

For SATO, GoPlus-style results should be interpreted carefully if they flag:

- mint capability;
- burn capability;
- variable supply;
- privileged minter;
- contract-held ETH;
- non-standard liquidity structure;
- router-based execution.

These signals are not automatically false, but they need protocol context.

The most important compatibility point is that SATO’s mint and burn permissions are not intended to be controlled by an owner wallet. They are routed through the verified `SatoHook` contract.

### TokenSniffer

TokenSniffer-style analysis may be useful for identifying common ERC-20 risk patterns.

However, SATO’s curve-based launch can differ from assumptions used by standard token scanners.

Potential areas requiring manual review include:

- whether minting is owner-controlled or hook-controlled;
- whether selling is blocked or simply subject to curve rules;
- whether LP-lock warnings apply to a curve-reserve model;
- whether fees are ERC-20 transfer taxes or curve execution fees;
- whether router interaction implies custody or only execution routing.

TokenSniffer output should be treated as a prompt for source-level review, not as a final protocol assessment.

### DEX Interfaces

DEX interfaces and charting tools may show liquidity, volume, routing, or price data from secondary markets.

This can be useful, but it should not be confused with the primary SATO curve reserve.

SATO can have two different market contexts:

| Market Context | Meaning |
|---|---|
| Primary curve path | Buy and sell through `SatoHook` |
| Secondary market path | Trade on external pools such as SATO/USDT |
| Router path | Direct curve execution through `SatoSwapRouter` |
| Charting interface | Market display layer, not monetary policy |

Secondary market liquidity does not define the curve reserve.

## Recommended Scanner Classification

When a scanner or reviewer supports custom notes, SATO should be described as:

| Field | Recommended Interpretation |
|---|---|
| Token type | Curve-issued ERC-20 |
| Issuance model | Bonding curve minting through `SatoHook` |
| Redemption model | Curve burn/redemption through `SatoHook` |
| Mint authority | Locked contract minter |
| Reserve holder | `SatoHook` |
| Execution router | `SatoSwapRouter` |
| ERC-20 transfer tax | Not part of the token contract |
| External LP dependency | Not required for primary curve issuance |
| Admin monetary control | Not part of the documented architecture |
| Upgradeability | Not part of the token contract model |

## Router Compatibility

`SatoSwapRouter` is a verified execution router.

Its role is to help users perform direct curve buys and sells through the Uniswap v4 PoolManager while passing the swapper address to `SatoHook` through hook data.

The router should not be classified as:

- the reserve holder;
- the source of mint authority;
- the curve pricing engine;
- an admin controller;
- a treasury wallet.

The router is part of the execution path, while monetary enforcement remains inside `SatoHook`.

## Curve Fee Compatibility

Some scanners may detect a fee and classify the token as taxed.

SATO’s curve fee should not be confused with an ERC-20 transfer tax.

| Fee Context | Interpretation |
|---|---|
| ERC-20 transfer | No token-level transfer tax in the ERC-20 contract |
| Curve buy | Fee applied inside `SatoHook` execution |
| Curve sell | Fee applied inside `SatoHook` execution |
| Secondary market swap | Depends on the external market route |
| Admin-adjustable token tax | Not part of the documented token model |

Scanner tools should distinguish between protocol execution fees and token transfer taxes.

## Liquidity Compatibility

Traditional token scanners often expect a standard DEX liquidity pool and an LP-token lock.

SATO’s primary issuance path does not depend on that model.

The bonding curve uses ETH reserve accounting inside `SatoHook`. The hook also rejects external liquidity additions to the primary curve pool.

| Scanner Expectation | SATO Architecture |
|---|---|
| Standard LP pair | Primary curve pool uses hook-controlled behavior |
| LP tokens locked | Not the primary security mechanism |
| Liquidity equals DEX pool balance | Curve reserve is held by `SatoHook` |
| LP removal risk | Must be analyzed through hook reserve logic instead |

Secondary liquidity pools may exist separately, but they are not the same as the primary curve reserve.

## Honeypot Compatibility

A honeypot warning should always be reviewed, but SATO’s verified architecture should be interpreted carefully.

SATO has protocol-level curve rules, including:

- exact-input swap requirement;
- same-block buy/sell cooldown;
- max single buy amount;
- reserve-based redemption;
- self-deprecation near curve exhaustion.

These rules are part of the curve design.

They should not be confused with an owner-controlled blacklist, pause function, hidden sell block, or arbitrary transfer restriction.

## Manual Review Checklist

Before accepting or rejecting a scanner warning, reviewers should check:

- Is the deployed ERC-20 source verified?
- Is the deployed hook source verified?
- Is the deployed router source verified?
- Is the ERC-20 minter locked to `SatoHook`?
- Does the ERC-20 contain blacklist logic?
- Does the ERC-20 contain pause logic?
- Does the ERC-20 contain transfer tax logic?
- Does the ERC-20 use an upgradeable proxy?
- Does `SatoHook` control curve mint and burn behavior?
- Does `SatoHook` hold the ETH reserve?
- Does `SatoSwapRouter` only route execution?
- Are secondary market warnings being confused with primary curve behavior?

## Suggested Public Scanner Note

The following short note may be used when scanner platforms allow project-provided context:

```text
SATO is a curve-issued ERC-20. Minting and burning are expected protocol behavior and are restricted to the verified SatoHook contract. The ETH curve reserve is held by SatoHook. SatoSwapRouter is a verified execution router for direct curve buy/sell calls and does not control issuance, reserve custody, or curve pricing. Scanner output should be interpreted together with the verified ERC-20, hook, and router source code.
```

## Risk Interpretation

This document does not claim that SATO has no risk. It only explains how scanner findings should be interpreted in the context of the verified contract architecture.

The main residual risks are not typical owner-control risks, but protocol-level and market-level risks such as custom hook logic, curve execution behavior, Uniswap v4 settlement dependency, secondary market liquidity, slippage, MEV, and the absence of an emergency upgrade or pause mechanism.

## Limitations

Scanner compatibility does not mean the protocol is risk-free.

Automated tools can miss important issues, and they can also misclassify intentional protocol behavior. This document only explains how scanner results should be interpreted in the context of SATO’s verified architecture.

Users should independently review deployed contracts, source code, on-chain state, market liquidity, and transaction behavior before relying on any scanner result.

## Related Documents

- `docs/security/01_Overview.md`
- `docs/security/02_Threat_Model.md`
- `docs/security/03_Trust_Model.md`
- `docs/security/04_Security_Guarantees.md`
- `docs/security/05_False_Positive_Analysis.md`
- `docs/protocol/03_ERC20.md`
- `docs/protocol/04_SatoHook.md`
- `docs/protocol/05_Bonding_Curve.md`
- `docs/protocol/06_Reserve_Model.md`
- `docs/research/04_Economic_Security.md`
- `docs/research/06_References.md`
