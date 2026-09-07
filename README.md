# Arc Transaction Observer

A developer tool for inspecting **Arc Testnet** transaction infrastructure in real time.

It checks the Arc RPC, validates the chain ID, reports the latest block and gas conditions, verifies Arc's USDC ERC-20 interface, and tracks confirmed transaction receipts. It also makes Arc's USDC accounting model explicit: **native USDC uses 18-decimal EVM units for gas/value math, while the ERC-20 interface uses 6 decimals for token balances and transfers**.

This project is intentionally read-only. It does not require a private key and never submits a transaction.

## Why this exists

Arc is EVM-compatible, but its USDC-native design introduces details that infrastructure and application developers need to handle correctly:

- Arc Testnet chain ID: `5042002`
- Primary RPC: `https://rpc.testnet.arc.network`
- USDC is the native gas asset
- Native value/gas math uses 18-decimal units
- The USDC ERC-20 interface uses 6 decimals
- Arc recommends an adequate EIP-1559 max fee; the observer exposes a conservative recommendation from live RPC data

The goal is to turn those assumptions into executable checks instead of leaving them as documentation-only knowledge.

## Features

### Live network health

`npm run health` produces a JSON snapshot containing:

- RPC endpoint and latency
- chain ID validation
- latest block number, hash and timestamp
- observed gas price
- recommended max fee per gas
- native/USDC decimal model
- USDC ERC-20 `decimals()` check against the Arc system address

### Transaction receipt tracker

Given an Arc Testnet transaction hash, the tracker polls for its receipt and reports:

- success or revert state
- block number, hash and timestamp
- gas used
- effective gas price
- transaction fee expressed as native USDC

```bash
npm run receipt -- 0x<transaction-hash>
```

### Tests and CI

`npm test` verifies the unit-conversion and fee-floor logic locally. GitHub Actions runs typechecking, all unit tests, and a live Arc Testnet RPC check on pushes and pull requests.

## Quick start

```bash
npm install
npm run build
npm test
npm run health
```

Optional custom RPC:

```bash
ARC_RPC_URL=https://rpc.testnet.arc.network npm run health
```

## Verified live check

The first CI run completed successfully on 2026-09-07. It validated Arc Testnet chain ID `5042002`, queried live block `60936172`, measured a 320 ms RPC round trip for the health snapshot, observed a 25 gwei gas price, and verified that the USDC system contract reports 6 ERC-20 decimals.

CI run: https://github.com/aarin081/smart-transaction-stack/actions/runs/34143833417

## Arc references

- Arc RPC endpoints: https://docs.arc.io/arc/references/rpc-endpoints
- Arc gas and fees: https://docs.arc.io/arc/references/gas-and-fees
- Arc developer article on USDC native/6-decimal interfaces: https://www.arc.io/blog/usdc-for-every-action-how-arc-simplifies-building-onchain

## Security

No wallet file, seed phrase, or private key is required. `.env` files and common key-file patterns are ignored by Git.

## Roadmap

- add a block-stream/latency sampler
- add fixture-based receipt tests
- add a small Arc Testnet contract deployment demo with explorer links
