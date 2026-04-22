# FundMe — Decentralized Crowdfunding Contract

> A production-style smart contract for decentralized fundraising on Ethereum, with real-time USD price enforcement via Chainlink oracles. Built and tested entirely with Foundry.

![Solidity](https://img.shields.io/badge/Solidity-^0.8.18-363636?logo=solidity)
![Foundry](https://img.shields.io/badge/Built%20with-Foundry-FFDB1C)
![Network](https://img.shields.io/badge/Network-Sepolia%20Testnet-blue)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Overview

FundMe is a decentralized crowdfunding contract that enforces a **minimum contribution of $5 USD** per funder, evaluated in real time using a **Chainlink Price Feed**. The contract owner can withdraw the accumulated ETH at any time, and the system is designed to be deterministic and fully testable — including against a live forked state of Sepolia.

This project demonstrates end-to-end smart contract development: design, Chainlink oracle integration, deployment scripting, and a full Foundry test suite.

**Key capabilities:**
- USD-denominated minimum contribution enforced on-chain via Chainlink ETH/USD price feed
- Owner-only withdrawal with access control
- Funder ledger tracking contributions per address
- Deployment scripts for both local Anvil and Sepolia testnet
- Mock price feed contract for isolated local testing

---

## Tech Stack

| Tool | Role |
|------|------|
| Solidity `^0.8.18` | Smart contract language |
| Foundry (Forge + Cast + Anvil) | Compile, test, deploy, interact |
| Chainlink Price Feeds | Real-time ETH/USD conversion |
| Sepolia Testnet | Live deployment environment |

---

## Technical Deep Dive

### Contract architecture

```
FundMe.sol
├── fund()              — accepts ETH, checks $5 USD minimum via Chainlink
├── withdraw()          — owner-only, resets funder ledger after transfer
├── getConversionRate() — converts ETH amount to USD using oracle price
└── getPriceFeed()      — returns the AggregatorV3Interface address in use

PriceConverter.sol (library)
└── getConversionRate() — stateless utility, imported by FundMe
```

### Chainlink integration

The contract reads from `AggregatorV3Interface.latestRoundData()` to fetch the current ETH/USD price. This price is used to convert incoming `msg.value` (in wei) to a USD equivalent before checking the minimum threshold. On testnet, the live Sepolia price feed address is used. In local tests, a `MockV3Aggregator` is deployed to simulate feed responses deterministically.

### Deployment scripting

Two deployment scripts handle environment-specific logic:

- `HelperConfig.s.sol` — detects the active chain ID and returns either the live Sepolia feed address or deploys a mock
- `DeployFundMe.s.sol` — uses HelperConfig to deploy FundMe with the correct feed, scriptable with `forge script`

### Test suite

Tests are written in Foundry and cover three layers:

| Test type | What it covers |
|-----------|----------------|
| Unit | fund(), withdraw(), access control, minimum amount revert |
| Integration | Full deploy → fund → withdraw flow via deployment scripts |
| Forked (Sepolia) | Live price feed interaction against a real network fork |

Run tests:
```bash
# Local unit + integration
forge test

# Forked against Sepolia
forge test --fork-url $SEPOLIA_RPC_URL
```

### Key design decisions

- **Library pattern** — `PriceConverter` is a Solidity library attached to `uint256`, keeping `FundMe.sol` clean and the conversion logic reusable
- **Mock injection** — local test environment injects a `MockV3Aggregator` rather than hitting a real feed, ensuring tests are fast and deterministic
- **Owner pattern** — `msg.sender` at deploy time is set as `i_owner` (immutable), reducing storage reads and gas cost on every `withdraw()` call

---

## Getting Started

```bash
git clone https://github.com/Ayisha-mohideen/foundry-fund-me
cd foundry-fund-me
forge install
forge build
```

Set up your environment:
```bash
cp .env.example .env
# Add SEPOLIA_RPC_URL and PRIVATE_KEY
```

Deploy to Sepolia:
```bash
forge script script/DeployFundMe.s.sol --rpc-url $SEPOLIA_RPC_URL --broadcast
```

---

## Project Status

Complete. Part of the [Cyfrin Updraft](https://updraft.cyfrin.io) Solidity smart contract developer curriculum.

---

*Built by [Ayisha](https://github.com/Ayisha-mohideen) · Cyfrin profile: [profiles.cyfrin.io/u/vivacious1574](https://profiles.cyfrin.io/u/vivacious1574)*
