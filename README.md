# Web3 Jackpot Casino Game on Solana | Provably Fair Blockchain Casino

[![Solana](https://img.shields.io/badge/Solana-9945FF?style=for-the-badge&logo=solana&logoColor=white)](https://solana.com)
[![Anchor](https://img.shields.io/badge/Anchor-0.30.1-blue?style=for-the-badge)](https://anchor-lang.com)
[![Rust](https://img.shields.io/badge/Rust-1.18.18-orange?style=for-the-badge&logo=rust)](https://www.rust-lang.org)
[![License: ISC](https://img.shields.io/badge/License-ISC-green.svg?style=for-the-badge)](https://opensource.org/licenses/ISC)

A **Web3 jackpot casino game** built on Solana: decentralized, provably fair, and secure. This repository provides the smart contract and CLI for a **blockchain casino jackpot** with verifiable random winner selection (ORAO VRF), instant SOL payouts, and full on-chain transparency.

**Keywords**: Web3 jackpot casino game, blockchain casino, Solana casino, decentralized jackpot, provably fair casino, Web3 gambling, crypto jackpot game.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Project Architecture](#-project-architecture)
- [How to Run This Project](#-how-to-run-this-project)
- [Key Features](#-key-features)
- [How It Works](#-how-it-works)
- [Prerequisites](#️-prerequisites)
- [Installation & Setup](#-installation--setup)
- [Quick Start (Build & Deploy)](#-quick-start-build--deploy)
- [Usage Guide](#-usage-guide)
- [Smart Contract Functions](#-smart-contract-functions)
- [Security](#-security)
- [Development](#-development)
- [Author & Contact](#-author--contact)
- [License](#-license)

---

## 🎯 Overview

**Web3 Jackpot Casino Game** is a fully decentralized casino jackpot system on the Solana blockchain using the Anchor framework. Operators create rounds with configurable duration, minimum bet, and max players; users deposit SOL to join; winners are chosen fairly via **ORAO VRF** (Verifiable Random Function). Ideal for:

- 🎲 **Web3 casino platforms** on Solana  
- 🏆 **Jackpot gaming dApps** with transparent winner selection  
- 💰 **DeFi / crypto gaming** requiring provably fair randomness  
- 🎰 **Blockchain casino games** with automated prize distribution  

### Why This Web3 Jackpot Casino?

- ✅ **Provably fair** — ORAO VRF for verifiable random winner selection  
- ✅ **Transparent** — All game logic on-chain, auditable  
- ✅ **Automated** — Smart contract handles rounds, deposits, and payouts  
- ✅ **Secure** — Anchor best practices, PDA-based design  
- ✅ **Low fees** — Solana’s high throughput and low cost  
- ✅ **Customizable** — Flexible round parameters per game mode  

---

## 🏗️ Project Architecture

### High-Level Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                     Web3 Jackpot Casino Game                     │
├─────────────────────────────────────────────────────────────────┤
│  CLI (TypeScript)     │  Smart Contract (Rust/Anchor)  │  ORAO  │
│  • config             │  • configure, create_game       │  VRF   │
│  • create / join      │  • join_game, set_winner         │        │
│  • winner / claim     │  • claim_reward                 │        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    Solana (devnet / mainnet)
```

### Repository Layout

```
.
├── programs/
│   └── jackpot_smart_contract/          # Rust program (Anchor)
│       ├── src/
│       │   ├── lib.rs                   # Program entry & instruction dispatch
│       │   ├── constants.rs             # PDA seeds (CONFIG, GAME_GROUND, GLOBAL)
│       │   ├── errors.rs                # Custom errors
│       │   ├── utils.rs                 # Helpers
│       │   ├── misc.rs                  # VRF state parsing
│       │   ├── state/
│       │   │   ├── mod.rs
│       │   │   ├── config.rs            # Global Config account
│       │   │   └── gameground.rs        # GameGround per round + DepositInfo
│       │   └── instructions/
│       │       ├── mod.rs
│       │       ├── admin/
│       │       │   ├── configure.rs      # Init/update global config
│       │       │   ├── create_game.rs    # Create jackpot round
│       │       │   └── set_winner.rs     # Consume ORAO VRF, set winner
│       │       └── user/
│       │           ├── join_game.rs     # Join round with SOL
│       │           └── claim_reward.rs  # Winner claims jackpot
│       └── Cargo.toml
├── cli/                                 # TypeScript CLI
│   ├── command.ts                       # Commander subcommands
│   ├── scripts.ts                       # config, create, join, winner, claim
│   └── utils.ts
├── lib/                                 # Shared TS (util, scripts, constants)
├── idl/                                 # Generated IDL (JSON + TS)
├── keys/                                # Wallet keypairs (gitignored)
├── Anchor.toml                          # Anchor config, program ID, provider
├── package.json                         # Node deps + "yarn script" entry
└── README.md
```

### On-Chain Data Model

| Account    | Purpose |
|-----------|---------|
| **Config** | Global config: `authority`, `payer_wallet`, `team_wallet`, `game_round`, `platform_fee`, `min_deposit_amount`, `max_joiner_count`, `initialized`. |
| **GameGround** (per round) | Round state: `creator`, `game_round`, `create_date`, `start_date`, `end_date`, `round_time`, `total_deposit`, `rand`, `winner`, `user_count`, `min_deposit_amount`, `max_joiner_count`, `force`, `is_completed`, `is_claimed`, `deposit_list` (user + amount). |
| **Global vault (PDA)** | Holds SOL for the jackpot; winner receives funds from here on `claim_reward`. |

### Game Flow (Architecture)

1. **Admin** initializes `Config` once, then creates rounds via `create_game` (round time, min deposit, max joiners).  
2. **Players** call `join_game` with SOL; round timer starts when the second player joins.  
3. When **round time expires**, admin calls `set_winner`; contract reads ORAO VRF result and selects winner by weighted probability (deposit / total_deposit).  
4. **Winner** calls `claim_reward` to receive the jackpot SOL from the global vault.  

---

## 🚀 How to Run This Project

End-to-end steps to run the Web3 jackpot casino locally and on devnet.

### 1. Prerequisites

Install and verify:

- **Rust** (1.75+): [rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)  
- **Solana CLI** (1.18+): [docs.solana.com/cli/install-solana-cli-tools](https://docs.solana.com/cli/install-solana-cli-tools)  
- **Anchor** 0.30.1: [anchor-lang.com/docs/installation](https://anchor-lang.com/docs/installation)  
- **Node.js** 16+ and **Yarn**

```bash
rustc --version
solana --version
anchor --version
node --version
yarn --version
```

### 2. Clone and Install

```bash
git clone https://github.com/yourusername/Solana-Casino-Jackpot-Smart-Contract.git
cd Solana-Casino-Jackpot-Smart-Contract
yarn install
```

### 3. Solana and Wallet Setup

```bash
solana config set --url devnet
solana config get

# Create keypair (e.g. for admin)
mkdir -p keys
solana-keygen new -o ./keys/admin.json
solana airdrop 5 $(solana-keygen pubkey ./keys/admin.json) -u devnet
```

Set `Anchor.toml` provider to use your keypair (e.g. `wallet = "./keys/admin.json"`).

### 4. Build the Program

```bash
RUSTUP_TOOLCHAIN="nightly-2024-11-19" anchor build
anchor keys sync
RUSTUP_TOOLCHAIN="nightly-2024-11-19" anchor build
```

### 5. Deploy

```bash
anchor deploy
```

Note the Program ID; it should match `declare_id!(...)` in `programs/jackpot_smart_contract/src/lib.rs` and `Anchor.toml`.

### 6. Initialize and Play (CLI)

```bash
# Initialize global config (admin)
yarn script config

# Create a round: 5 min, 0.1 SOL min, 100 max players
yarn script create -t 300 -d 100000000 -j 100

# Join round 0 with 0.5 SOL (use a player keypair via -k if needed)
yarn script join -a 500000000 -g 0

# After round ends, admin sets winner (uses ORAO VRF)
yarn script winner -g 0

# Winner claims
yarn script claim -g 0
```

Optional CLI options: `-e devnet`, `-r <RPC_URL>`, `-k <keypair_path>`.

**You’ve now run the full Web3 jackpot casino flow: config → create → join → set_winner → claim.**

---

## ✨ Key Features

- **Multi-round jackpot**: Unlimited rounds with configurable time, min deposit, max joiners.  
- **Provably fair winner**: ORAO VRF; each player’s chance = deposit / total pool.  
- **Instant payouts**: Winner claims SOL from the global vault.  
- **Admin controls**: Configure, create rounds, set winner (admin-only).  
- **TypeScript CLI**: Full control via `yarn script` (config, create, join, winner, claim).  

---

## 🎲 How It Works

1. **Admin** creates a round → players join with SOL.  
2. Round **countdown** starts when the 2nd player joins.  
3. When **time expires**, admin calls `set_winner`; contract uses ORAO VRF to pick winner.  
4. **Winner** calls `claim_reward` → SOL sent to winner’s wallet.  

---

## 🛠️ Prerequisites

- Rust 1.75+  
- Solana CLI 1.18+  
- Anchor 0.30.1  
- Node.js 16+  
- Yarn  

---

## 📦 Installation & Setup

Same as [How to Run This Project](#-how-to-run-this-project): clone → `yarn install` → configure Solana CLI and wallet → build → deploy → `yarn script config`.

---

## ⚡ Quick Start (Build & Deploy)

```bash
yarn install
solana config set --url devnet
RUSTUP_TOOLCHAIN="nightly-2024-11-19" anchor build
anchor keys sync && RUSTUP_TOOLCHAIN="nightly-2024-11-19" anchor build
anchor deploy
yarn script config
```

---

## 📖 Usage Guide

| Role   | Command example |
|--------|------------------|
| Admin  | `yarn script config` |
| Admin  | `yarn script create -t 300 -d 100000000 -j 100` |
| Player | `yarn script join -a 500000000 -g 0` |
| Admin  | `yarn script winner -g 0` |
| Winner | `yarn script claim -g 0` |

Parameters: `-t` round time (seconds), `-d` min deposit (lamports), `-j` max joiners, `-a` join amount (lamports), `-g` round number.

---

## 🔧 Smart Contract Functions

- **configure** — Init/update global config (admin).  
- **create_game** — Create jackpot round (admin).  
- **set_winner** — Read VRF, set winner, mark round completed (admin).  
- **join_game** — Join round with SOL (any wallet).  
- **claim_reward** — Transfer jackpot to winner (winner only).  

---

## 🔒 Security

- Admin-only functions gated by `Config.authority`.  
- PDAs for config, game grounds, and global vault.  
- ORAO VRF for randomness; no central manipulation.  
- Safe math and state checks. Use a secure keypair and audit before mainnet.  

---

## 👨‍💻 Development

- **Build**: `RUSTUP_TOOLCHAIN="nightly-2024-11-19" anchor build`  
- **Test**: `anchor test`  
- **Lint**: `yarn lint` / `yarn lint:fix`  

---

## 👤 Author & Contact

**Telegram**: [@microRustyme](https://t.me/microRustyme) (ID: microRustyme)  

For questions, support, or collaboration related to this Web3 jackpot casino game, reach out via Telegram.

---
