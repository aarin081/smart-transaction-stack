# Arc Transaction Observer

A small developer tool for inspecting **Arc Testnet** transaction infrastructure in real time.

It checks the Arc RPC, validates the chain ID, reports the latest block and gas conditions, and makes Arc's USDC accounting model explicit: **native USDC uses 18-decimal EVM units for gas/value math, while the ERC-20 interface uses 6 decimals for token balances and transfers**.

This project is intentionally read-only. It does not require a private key and never submits a transaction.

## Why this exists

Arc is EVM-compatible, but its USDC-native design introduces a few details that infrastructure and application developers need to handle correctly:

- Arc Testnet chain ID: `5042002`
- Primary RPC: `https://rpc.testnet.arc.network`
- USDC is the native gas asset
- Native value/gas math uses 18-decimal units
- The USDC ERC-20 interface uses 6 decimals
- Arc recommends an adequate EIP-1559 max fee; the observer exposes a conservative recommendation from live RPC data

The goal is to turn those assumptions into checks instead of leaving them as documentation-only knowledge.

## What it does

`npm run health` produces a live JSON snapshot containing:

- RPC endpoint and latency
- chain ID validation
- latest block number and timestamp
- observed gas price
- recommended max fee per gas
- native/USDC decimal model
- USDC ERC-20 `decimals()` check against the Arc system address

`npm test` verifies the unit-conversion and fee-floor logic locally.

## Quick start

```bash
npm install
npm test
npm run health
```

Optional custom RPC:

```bash
ARC_RPC_URL=https://rpc.testnet.arc.network npm run health
```

## Example output shape

```json
{
  "network": "Arc Testnet",
  "chainId": 5042002,
  "chainIdValid": true,
  "latestBlock": 0,
  "rpcLatencyMs": 0,
  "observedGasPriceGwei": "0",
  "recommendedMaxFeeGwei": "20",
  "usdc": {
    "nativeDecimals": 18,
    "erc20Decimals": 6,
    "erc20DecimalsCheck": 6
  }
}
```

## Arc references

- Arc RPC endpoints: https://docs.arc.io/arc/references/rpc-endpoints
- Arc gas and fees: https://docs.arc.io/arc/references/gas-and-fees
- Arc developer article on USDC native/6-decimal interfaces: https://www.arc.io/blog/usdc-for-every-action-how-arc-simplifies-building-onchain

## Security

No wallet file, seed phrase, or private key is required. `.env` files and common key-file patterns are ignored by Git.

## Status

Early developer-tooling prototype. The next milestone is transaction-receipt/finality tracking and a small Arc Testnet deployment demo.