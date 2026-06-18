# Monetary Model

## Overview

SATO implements a deterministic monetary protocol based entirely on immutable smart contract execution.

Unlike conventional ERC-20 assets, token issuance is not controlled by a treasury, multisignature wallet, governance proposal, or administrative authority.

Instead, monetary supply evolves exclusively through protocol-defined mint and burn operations.

---

## Design Philosophy

The monetary model follows one fundamental principle:

> Monetary policy should be deterministic, transparent, and enforceable entirely by code.

Every unit of supply is created or removed according to immutable protocol rules.

No participant possesses discretionary authority over issuance.

---

## Issuance

New SATO enters circulation only through protocol mint operations.

Minting requires ETH to be deposited into the protocol reserve.

The protocol calculates issuance using its deterministic bonding curve.

No privileged account can mint tokens outside protocol rules.

---

## Redemption

Token holders may redeem SATO through protocol burn operations.

Burning permanently removes tokens from circulation while returning ETH according to the reserve model.

The redemption process follows the same deterministic protocol rules used for issuance.

---

## Reserve Backing

Every mint operation contributes ETH to the protocol reserve.

Every burn operation withdraws ETH according to the protocol calculation.

Reserve accounting is performed entirely on-chain and remains publicly verifiable.

---

## Deterministic Supply

Supply evolution depends only on protocol interactions.

The protocol contains no discretionary monetary expansion.

Supply cannot be altered by governance decisions or administrative intervention.

---

## Summary

The monetary model replaces human monetary policy with deterministic protocol execution.

Issuance, redemption, reserve accounting, and supply evolution are enforced directly by immutable smart contracts.
