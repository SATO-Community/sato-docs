# False Positive Analysis

This document explains how automated token scanners may misclassify certain SATO protocol features as risks when those features are interpreted without the full contract architecture.

Scanner output can be useful as an initial signal, but it should not be treated as a substitute for source-level contract review.

## Scope

This analysis covers the main contracts and components relevant to scanner interpretation:

| Component | Address / Role |
|---|---|
| SATO ERC-20 | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| SatoHook | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` |
| SatoSwapRouter | `0x06A645079cd4F3Bb38FfaD47f92180B8041145E3` |
| Uniswap v4 PoolManager | External settlement layer |
| Curve Reserve | ETH held by `SatoHook`, excluding accrued fees |

## Scanner Interpretation Principle

SATO uses a non-standard issuance model.

Instead of a fixed initial token supply or owner-managed minting, SATO is minted and burned through a bonding curve implemented inside `SatoHook`.

Because of this, automated scanners may detect minting, burning, variable supply, ETH custody, or router interaction and classify them as suspicious. These signals require protocol-specific interpretation.

## Common False Positives

### 1. Mint Function Detected

A scanner may flag the token because the ERC-20 contract contains a `mint` function.

This is expected behavior.

SATO has no premine. New tokens are minted only when users buy through the bonding curve. The ERC-20 token contract does not allow arbitrary public minting.

Minting is restricted to the locked minter address, which is `SatoHook`.

| Item | Interpretation |
|---|---|
| Scanner signal | Mint function exists |
| Protocol meaning | Curve-based issuance |
| Risk if misread | May appear as owner-controlled inflation |
| Actual constraint | Only the locked hook minter can mint |

The relevant distinction is that minting is not controlled by an owner wallet. It is tied to the curve execution path.

## 2. Burn Function Detected

A scanner may flag the token because the ERC-20 contract contains a `burn` function.

This is also expected behavior.

Burning is part of the redemption model. When users sell back into the bonding curve, SATO is burned and ETH is released from the hook reserve according to the curve formula.

| Item | Interpretation |
|---|---|
| Scanner signal | Burn function exists |
| Protocol meaning | Redemption through the curve |
| Risk if misread | May appear as arbitrary token destruction |
| Actual constraint | Burn is executed through the hook-controlled sell path |

The burn function does not create an admin ability to destroy arbitrary user balances.

## 3. Privileged Minter Address

A scanner may detect that one address has minting authority.

This is accurate, but incomplete without context.

The ERC-20 token contract uses a minter role, but the minter is locked to the `SatoHook` contract. The purpose of this design is to separate the token ledger from the monetary logic.

| Role | Meaning |
|---|---|
| ERC-20 token | Records balances and total supply |
| SatoHook | Controls curve minting and burning |
| Deployer | Can only set the minter once |
| User wallet | Cannot mint directly |

Once the minter is set, mint and burn authority belongs to the hook contract, not a general admin wallet.

## 4. Supply Can Change

A scanner may classify SATO as risky because total supply is not fixed.

This is expected.

SATO supply changes through mint and burn operations. The supply expands when ETH enters the curve and contracts when SATO is redeemed back into the curve.

This does not mean the supply can be arbitrarily changed by an owner.

| Supply Change | Source |
|---|---|
| Increase | Curve buy through `SatoHook` |
| Decrease | Curve sell through `SatoHook` |
| Admin mint | Not part of the protocol model |
| Admin burn | Not part of the protocol model |

Variable supply is a core monetary feature, not automatically an owner-control risk.

## 5. ETH Reserve Balance

A scanner may detect that the hook contract holds ETH.

This is expected.

The ETH reserve is held by `SatoHook`. The reserve backs curve redemptions. Users who sell into the curve receive ETH from the hook according to the bonding curve calculation.

However, the hook balance should not be interpreted as a generic treasury balance.

The curve reserve is:

`curve reserve = hook ETH balance - accrued fees`

| Item | Interpretation |
|---|---|
| Hook ETH balance | ETH held by the hook contract |
| Fees accrued | Protocol fees retained in the hook |
| Curve reserve | ETH available according to the reserve model |
| Owner withdrawal | Not part of the documented reserve model |

The reserve should be evaluated together with the hook source code, not only by checking whether the contract holds ETH.

## 6. Liquidity Lock Warnings

Some scanners are designed around standard ERC-20 liquidity-pool launches. They may expect liquidity to be added to a DEX pool and locked.

SATO’s primary issuance path is different.

The bonding curve does not rely on an externally supplied LP position for minting and redemption. The curve reserve is held by `SatoHook`, and external liquidity additions to the curve pool are forbidden by the hook.

| Scanner Assumption | SATO Design |
|---|---|
| Token launches with LP liquidity | SATO launches through curve issuance |
| LP tokens should be locked | Curve pool does not use external LP liquidity |
| Liquidity lock proves safety | Reserve behavior must be checked in hook code |

Secondary markets may exist separately, but they are not the same as the primary curve reserve.

## 7. Router Interaction

A scanner or reviewer may detect interaction with `SatoSwapRouter`.

This should not be interpreted as router custody or router control over monetary policy.

`SatoSwapRouter` is a minimal execution router. It helps route direct curve buys and sells through the Uniswap v4 PoolManager and passes the swapper address to `SatoHook` through hook data.

| Component | Role |
|---|---|
| SatoSwapRouter | Execution routing |
| PoolManager | Uniswap v4 settlement |
| SatoHook | Curve logic, mint/burn, reserve custody |
| SATO ERC-20 | Token ledger |

The router does not define the curve, hold the reserve, or gain independent minting authority.

## 8. Owner / Admin Risk Flags

Automated scanners may search for owner-style controls such as privileged minting, pausing, blacklist logic, transfer restrictions, upgradeability, or fee changes.

The SATO ERC-20 token does not implement typical owner-controlled restriction features such as:

| Feature | Status |
|---|---|
| Transfer tax | Not present in ERC-20 transfers |
| Blacklist | Not present |
| Pause | Not present |
| Upgradeable proxy | Not present |
| Owner-controlled mint | Not present after minter lock |
| Owner-controlled burn | Not present |

The protocol should still be reviewed at the hook and router level, but common ERC-20 owner-control warnings should be interpreted carefully.

## 9. Honeypot or Sell Restriction Warnings

Some scanners may flag non-standard sell behavior.

SATO does include protocol-level sell rules in the curve path, but these are not the same as an owner-controlled honeypot.

Relevant curve-side constraints include:

| Constraint | Purpose |
|---|---|
| Same-block buy/sell cooldown | Reduces immediate cycling |
| Exact-input swap requirement | Keeps hook accounting deterministic |
| Reserve-based redemption | Ensures sells follow the curve model |
| Self-deprecation near exhaustion | Stops further curve minting while preserving burn behavior |

These constraints are part of the monetary design and should be evaluated in source context.

A scanner warning should not be ignored, but it should be checked against the verified hook logic.

## 10. Fee or Tax Misclassification

SATO curve buys and sells include a curve-level fee.

This should not be confused with an ERC-20 transfer tax.

The ERC-20 token itself does not apply a tax on normal token transfers. The fee applies to curve mint and burn execution inside `SatoHook`.

| Fee Type | Status |
|---|---|
| ERC-20 transfer tax | Not present |
| Curve buy/sell fee | Present |
| Admin-adjustable tax | Not part of the documented model |
| Hidden transfer restriction | Not present in the ERC-20 token |

Scanner tools may not distinguish between a protocol execution fee and a token-level transfer tax.

## Verification Checklist

A reviewer should verify scanner findings against the following contract-level facts.

### ERC-20 Token

- Confirm the token contract is verified.
- Confirm the token name and symbol are `sato`.
- Confirm mint and burn are restricted to the minter.
- Confirm the minter is locked to `SatoHook`.
- Confirm there is no blacklist logic.
- Confirm there is no pause logic.
- Confirm there is no transfer tax logic.
- Confirm there is no upgradeable proxy pattern.

### SatoHook

- Confirm the hook contract is verified.
- Confirm the hook validates the expected ETH/SATO pool.
- Confirm external liquidity additions to the curve pool are forbidden.
- Confirm buys mint through the curve.
- Confirm sells burn through the curve.
- Confirm ETH reserve accounting separates accrued fees from reserve balance.
- Confirm same-block cooldown and max-buy rules are enforced by code.
- Confirm self-deprecation affects further minting, not redemption.

### SatoSwapRouter

- Confirm the router contract is verified.
- Confirm the router is an execution helper.
- Confirm the router does not hold the curve reserve as its monetary role.
- Confirm the router does not define mint, burn, or curve pricing rules.
- Confirm monetary enforcement remains inside `SatoHook`.

## Scanner Platforms

Scanner tools such as GoPlus Security and TokenSniffer can be useful for quick public risk screening.

However, their output should be interpreted together with verified contract source code.

| Tool Type | Useful For | Limitation |
|---|---|---|
| Automated scanner | Quick detection of common patterns | May misread custom protocol architecture |
| Source review | Understanding actual permissions and flows | Requires manual technical review |
| On-chain verification | Checking deployed bytecode and state | Does not by itself explain design intent |
| Audit | Independent security assessment | Separate from scanner output |

Scanner findings should be treated as prompts for review, not final conclusions.

## Reviewer Summary

The most likely false positives for SATO are:

- Mint function detected
- Burn function detected
- Variable supply detected
- Minter role detected
- Contract holds ETH
- No traditional LP lock detected
- Router interaction detected
- Sell behavior differs from standard ERC-20 trading assumptions
- Curve fee misread as transfer tax

These signals are not meaningless, but they require architectural context.

SATO’s core risk analysis should focus on the verified ERC-20 token, `SatoHook`, curve math, reserve accounting, Uniswap v4 PoolManager interaction, and the verified router execution path.

## Limitations

This document is not an audit.

It does not prove that the protocol is risk-free. It only explains how scanner findings should be interpreted in the context of SATO’s verified architecture.

Users and reviewers should independently verify contract source code, deployed addresses, on-chain state, and transaction behavior before relying on any security conclusion.

## Related Documents

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
