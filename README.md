# Computer blockchain system services section

<!-- 561-group-standards:badges:start -->
[![Red Cup Engineering](https://img.shields.io/badge/561%20Group-Red%20Cup%20Engineering-c0392b)](https://github.com/red-cup-engineering)
[![License: MIT](https://img.shields.io/badge/license-MIT-c0392b)](LICENSE)
[![Indigenous data: CARE](https://img.shields.io/badge/Indigenous%20data-CARE-4b6f44)](https://github.com/red-cup-engineering/.github/blob/main/GOVERNANCE.md)
[![Tips welcome](https://img.shields.io/badge/tips-welcome-d4a017)](https://github.com/561-group/.github/blob/main/TIPS.md)
<!-- 561-group-standards:badges:end -->

Parent home under Red Cup Engineering for every chain-specific blockchain
service the Union operates. Union enterprises that need blockchain/EVM
capability have a proper seat to offer and consume it from here, rather than
each hand-rolling its own.

## Holdings

- `services/ethereum-engineering-services-section` — Ethereum/EVM chain services, the
  first chain-specific section seated here.
- `services/cross-chain-services-section` — chain registry, multi-fork harness and deterministic
  deployment for Tidepool across EVM chains, Hive and Solana.

Chain-specific work seats under `services/` as its own section. Shared
non-chain-specific blockchain code, if any accumulates, seats here directly.

## Shared data core

Chain-specific sections build on the Union's cryptographic data core
at `computer-data-engineering-services-section/services/computer-data-cryptography-services-section`
(`secp256k1`, `keccak256`, and EVM identity/signing primitives) rather than
re-deriving key math. The chain orders and transports commitments; it does not
own their data semantics.

## Org

- **Over**: red-cup-engineering (Internal Market)
- **Owns**: services/ethereum-engineering-services-section, services/cross-chain-services-section
