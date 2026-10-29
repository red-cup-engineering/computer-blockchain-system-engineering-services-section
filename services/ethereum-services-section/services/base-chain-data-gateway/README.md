# Base chain-data gateway

An x402 v2 seller for three keyless-public-RPC Base network chain-data
reads, deployed as one Cloudflare Worker:

- `GET /base/balance?address=0x...` -- native ETH balance (`eth_getBalance`).
- `GET /base/gas-oracle` -- current network gas price (`eth_gasPrice`).
- `GET /base/transaction?hash=0x...` -- one transaction by hash (`eth_getTransactionByHash`).
- `GET /healthz` -- unpriced liveness check.

Each read is proxied from a free, keyless public Base JSON-RPC endpoint
(`BASE_RPC_URL`, default `https://mainnet.base.org`); no paid RPC provider
is used. The HTTP payment boundary is the official `@x402/*` SDK reached
through `@red-cup-engineering/x402-services-section`'s `./fetch` export
(`createExactEvmFetchPaymentBoundary`), the transport built specifically
for a stateless fetch-handler host such as Cloudflare Workers -- the same
protocol core the house's Express-based sellers
(`metered-inference-tollbooth`, `paid-ephemeris-gateway`) use, with no
Express dependency.

## Configuration

Every settlement-affecting value is deployment-supplied (`wrangler.toml`
`[vars]` for public values, `wrangler secret put` for anything sensitive),
never hardcoded in `src/`, matching the env-var convention already live in
`metered-inference-tollbooth` and `paid-ephemeris-gateway`:

- `X402_NETWORK`: explicit EVM CAIP-2 chain. Default `eip155:84532` (Base
  Sepolia testnet). The live deployment sets `eip155:8453` for Base mainnet
  production.
- `X402_PAY_TO`: this deployment's receiving EVM address. **Required, no
  default** -- deciding which wallet receives real settlement is an owner
  custody decision, never inferred or invented by this package.
- `X402_ASSET`: the settlement asset's EVM address. Defaults to Circle's
  official USDC contract for the selected network (Base mainnet
  `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`, Base Sepolia
  `0x036CbD53842c5426634e7929541eC2318f3dCF7e`, verified against
  developers.circle.com/stablecoins/usdc-contract-addresses).
- `X402_FACILITATOR`: contracted x402 facilitator URL. Defaults to
  Coinbase's free, keyless public facilitator
  (`https://x402.org/facilitator`), which supports `eip155:84532` but
  **not** `eip155:8453` (its own `GET /supported` names which networks it
  serves). The mainnet deployment sets this
  explicitly to `https://facilitator.payai.network` -- a free, keyless
  third-party facilitator, protocol-assayed (schema-valid `/supported`
  declaring Base-mainnet `exact`, structured protocol-conformant refusal
  of malformed `/verify`/`/settle` payloads, and refusal of a
  recipient-mismatched probe) by
  `x402-services-section/scripts/assay-x402-facilitator.mjs`. No Coinbase
  CDP account or credentials are required.
- `BASE_RPC_URL`: the read RPC endpoint. Default `https://mainnet.base.org`
  (free, keyless). Reads are independent of `X402_NETWORK`.
- `X402_BALANCE_PRICE_ATOMIC`, `X402_GAS_ORACLE_PRICE_ATOMIC`,
  `X402_TRANSACTION_PRICE_ATOMIC`: per-route price in the asset's atomic
  units. Defaults `1000` / `1000` / `2000` (USDC has 6 decimals, so
  $0.001 / $0.001 / $0.002), matching the 402index.io comparables (Base
  Gas Oracle $0.0010/call, Base Transaction Check $0.0020/call).

## Base-mainnet settlement

This worker deploys to the custom domain
`base-chain.561.group` on Base mainnet (`eip155:8453`), settling through
`https://facilitator.payai.network` -- a free, keyless third-party
facilitator requiring no Coinbase CDP account -- to `X402_PAY_TO` (a
`wrangler secret put` value, an owner-custodied mainnet address; see the
`x402.payto-decision` record in `x402-services-section/data/logs/2026/09/01/`).
It is listed on [402index.io](https://402index.io/) (`chain-data` category).
Whether the deployed routes are actually answering, and what a probe of them
returns, is checked live with `wrangler`/`curl` against
`base-chain.561.group`, per the `whats-live` skill, never asserted here.

`wrangler.toml` binds only the `base-chain.561.group` custom-domain
route and declares no `workers_dev` binding, so the
`base-chain-data-gateway.morgan-sennhauser.workers.dev` host carries no
route for this worker; the custom domain is the only host this worker's
configuration serves.

`X402_NETWORK=eip155:84532` (Base Sepolia testnet) remains the package
default for local development and testing; production sets it explicitly
to `eip155:8453`.

## Tests

`npm test` runs `node --test` over the handler logic (route dispatch,
input validation, response shaping) with the RPC call mocked -- no
network access required.

## Thread

Five-custody thread (org-layering.md, 2026-09-01):
- science: [ethereum-services-section](../../../../../../../../../../../science-services-section/services/engineering-science-services-section/services/computer-science-services-section/services/blockchain-services-section/services/ethereum-services-section) — `content/text/glossary/terms/ethereum-virtual-machine.md` grounds Base (an EVM L2) as the machine model this gateway reads chain data from.
- engineering: **this section itself** — `src/base-reads.mjs` and `src/worker.mjs`, composed from `@red-cup-engineering/ethereum-services-section` (`public-evm-ledger`'s `jsonRpc`) and `@red-cup-engineering/x402-services-section` (`./fetch`'s `createExactEvmFetchPaymentBoundary`), both consumed as exact-pinned registry dependencies, with no hand-rolled JSON-RPC or payment-boundary code in this package.
- operations: [capability-market-services-section](../../../../../../../../../../../operations-services-section/services/union-services-section/services/union-market-services-section/services/capability-market-services-section) — names `x402-services-section` explicitly as the priced HTTP boundary/settlement leg of the Union's `RMN demand -> ... -> x402 clearing -> ...` capability-market pipeline this gateway's sales flow through.
- 561-group: [bare-cedar-fog-cloudflare-services-section](../../../../../../../../../../../561-group-services-section/services/561-group-computer-services-section/services/computer-machine-services-section/services/persistent-computer-machine-services-section/services/bare-cedar-fog-services-section/services/bare-cedar-fog-web-services-section/bare-cedar-fog-cloudflare-services-section) — the Cloudflare account custody this Worker deploys under (custom domain `base-chain.561.group`, Base mainnet).
- public: [public-blockchain-services-section](../../../../../../../../../../../public-services-section/services/public-computer-services-section/services/public-blockchain-services-section) — the house's general public blockchain custody (alongside `blockchain-trashpanda-services-section`); this product's public face is external, on [402index.io](https://402index.io/) (`chain-data` category, see this section's own `402index-listing.json`), named as a debt to eventually mirror into public custody rather than invented as a full link here.

## Entanglement debts

- public custody debt (above): this gateway's 402index.io listing has no
  mirror inside `public-blockchain-services-section`.
