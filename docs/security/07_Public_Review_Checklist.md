# Public Review Checklist

This checklist is intended for public reviewers, listing platforms, wallets, scanners, explorers, and community members reviewing the SATO protocol.

It is not a security audit. It is a structured review guide for verifying deployed contracts, documentation, scanner findings, and public protocol claims.

## 1. Basic Identity

- [ ] Confirm the token name is `sato`.
- [ ] Confirm the token symbol is `sato`.
- [ ] Confirm the token is deployed on Ethereum Mainnet.
- [ ] Confirm the ERC-20 token address is `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09`.
- [ ] Confirm the official website is `https://sat0.org`.
- [ ] Confirm the whitepaper is available at `https://sat0.org/whitepaper`.
- [ ] Confirm the documentation repository is `https://github.com/SATO-Community/sato-docs`.
- [ ] Confirm the asset repository is `https://github.com/SATO-Community/sato-assets`.

## 2. Deployed Contract Verification

Reviewers should verify the main deployed contracts directly on Etherscan.

| Component | Address |
|---|---|
| SATO ERC-20 | `0x829f4B62EEBE12Af653b4dD4fFc480966F7d7f09` |
| SatoHook | `0x0000f07d2B5F1Ddf3244b8780F972f306EFd2888` |
| SatoSwapRouter | `0x06A645079cd4F3Bb38FfaD47f92180B8041145E3` |
| Uniswap v4 PoolManager | `0x000000000004444c5dc75cB358380D2e3dE08A90` |

Checklist:

- [ ] Confirm the SATO ERC-20 contract source is verified.
- [ ] Confirm the `SatoHook` contract source is verified.
- [ ] Confirm the `SatoSwapRouter` contract source is verified.
- [ ] Confirm the reviewed addresses match the documentation.
- [ ] Confirm no lookalike token address is being used.
- [ ] Confirm no unofficial router or frontend is being treated as canonical.

## 3. ERC-20 Token Review

The ERC-20 token should be reviewed as the token accounting layer.

Checklist:

- [ ] Confirm the token contract manages balances and transfers.
- [ ] Confirm minting is restricted to the protocol minter.
- [ ] Confirm burning is restricted to the protocol minter.
- [ ] Confirm the minter is locked to `SatoHook`.
- [ ] Confirm there is no ERC-20 transfer tax.
- [ ] Confirm there is no blacklist function.
- [ ] Confirm there is no pause function.
- [ ] Confirm there is no upgradeable token proxy.
- [ ] Confirm there is no owner-controlled arbitrary minting after minter lock.
- [ ] Confirm there is no owner-controlled arbitrary burning.

## 4. SatoHook Review

`SatoHook` is the main monetary execution contract.

Checklist:

- [ ] Confirm `SatoHook` controls curve minting.
- [ ] Confirm `SatoHook` controls curve burning.
- [ ] Confirm `SatoHook` holds the ETH reserve.
- [ ] Confirm external liquidity additions to the curve pool are rejected.
- [ ] Confirm the hook validates the expected ETH/SATO pool configuration.
- [ ] Confirm curve buys follow the bonding-curve logic.
- [ ] Confirm curve sells follow the bonding-curve logic.
- [ ] Confirm reserve accounting separates accrued fees from the curve reserve.
- [ ] Confirm same-block buy/sell cooldown behavior is documented.
- [ ] Confirm maximum single-buy behavior is documented.
- [ ] Confirm self-deprecation behavior is documented.

## 5. Router Review

`SatoSwapRouter` is an execution router, not a monetary authority.

Checklist:

- [ ] Confirm `SatoSwapRouter` is verified.
- [ ] Confirm the router is used for direct curve buy and sell execution.
- [ ] Confirm the router does not hold the curve reserve as its monetary role.
- [ ] Confirm the router does not define curve pricing.
- [ ] Confirm the router does not have independent mint authority.
- [ ] Confirm the router does not replace `SatoHook` as the protocol enforcement layer.

## 6. Reserve Review

The ETH reserve should be interpreted through `SatoHook`.

Checklist:

- [ ] Confirm ETH reserve custody is handled by `SatoHook`.
- [ ] Confirm the router is not the reserve holder.
- [ ] Confirm the reserve is not described as an off-chain treasury.
- [ ] Confirm reserve accounting is documented in `docs/protocol/06_Reserve_Model.md`.
- [ ] Confirm direct ETH transfers are interpreted carefully.
- [ ] Confirm accrued fees are distinguished from the curve reserve.

## 7. Scanner Review

Automated scanners may produce false positives because SATO uses non-standard curve issuance.

Checklist:

- [ ] Review mint warnings in the context of curve issuance.
- [ ] Review burn warnings in the context of curve redemption.
- [ ] Review variable supply warnings in the context of bonding-curve supply changes.
- [ ] Review ETH balance warnings in the context of hook reserve custody.
- [ ] Review liquidity-lock warnings in the context of curve reserve design.
- [ ] Review router warnings in the context of execution routing.
- [ ] Confirm scanner output is not treated as a substitute for source-level review.

## 8. Market and Liquidity Review

SATO may have both primary curve activity and secondary market activity.

Checklist:

- [ ] Distinguish primary curve buys and sells from secondary market swaps.
- [ ] Confirm secondary market price does not define the curve formula.
- [ ] Confirm secondary market liquidity does not define the curve reserve.
- [ ] Review slippage before executing transactions.
- [ ] Consider MEV and market volatility risks.
- [ ] Confirm users are interacting with the intended route.

## 9. Documentation Review

Reviewers should check the documentation set as a whole.

Recommended documents:

- [ ] `docs/listing/01_Token_Information.md`
- [ ] `docs/protocol/01_Protocol_Overview.md`
- [ ] `docs/protocol/02_Architecture.md`
- [ ] `docs/protocol/03_ERC20.md`
- [ ] `docs/protocol/04_SatoHook.md`
- [ ] `docs/protocol/05_Bonding_Curve.md`
- [ ] `docs/protocol/06_Reserve_Model.md`
- [ ] `docs/security/01_Overview.md`
- [ ] `docs/security/02_Threat_Model.md`
- [ ] `docs/security/03_Trust_Model.md`
- [ ] `docs/security/04_Security_Guarantees.md`
- [ ] `docs/security/05_False_Positive_Analysis.md`
- [ ] `docs/security/06_Scanner_Compatibility.md`
- [ ] `docs/research/05_Formal_Reasoning.md`
- [ ] `docs/research/06_References.md`

## 10. Red Flags to Investigate

The following should be investigated carefully if encountered:

- [ ] A different token address is being promoted.
- [ ] A different router is being presented as official without documentation.
- [ ] A social account claims privileged authority over SATO.
- [ ] A scanner result is presented without source-level context.
- [ ] A frontend routes users through an unknown contract.
- [ ] A listing uses an incorrect logo, website, or contract address.
- [ ] A claim says SATO is risk-free or audited when no formal audit is provided.
- [ ] A claim says the router controls issuance or reserve custody.
- [ ] A claim says secondary market liquidity is the same as the curve reserve.

## 11. Review Conclusion Template

Reviewers may use the following summary format:

```text
Reviewed:
- ERC-20 token address
- SatoHook address
- SatoSwapRouter address
- Contract source verification
- Token mint/burn authorization
- Reserve custody model
- Scanner interpretation
- Documentation consistency

Conclusion:
SATO should be interpreted as a curve-issued ERC-20 protocol where minting, burning, and reserve behavior are enforced through the verified SatoHook contract. SatoSwapRouter is an execution router, not the monetary authority. Scanner warnings should be reviewed in protocol context and should not be treated as final conclusions without source-level verification.
```

## Limitations

This checklist is informational.

It does not prove that SATO is risk-free. It does not replace an independent smart contract audit, formal verification process, economic risk review, or transaction-level due diligence.

Users and reviewers should independently verify deployed contracts, source code, on-chain state, market liquidity, transaction behavior, and scanner output before relying on any conclusion.
