<p align="center">
  <img src="https://img.shields.io/badge/SoDEX-Volume%20Bot-blueviolet?style=for-the-badge&logo=ethereum&logoColor=white" alt="SoDEX Bot"/>
  <img src="https://img.shields.io/badge/version-5.8.1-blue?style=for-the-badge" alt="Version"/>
  <img src="https://img.shields.io/badge/node-%3E%3D18.0-brightgreen?style=for-the-badge&logo=node.js&logoColor=white" alt="Node"/>
  <img src="https://img.shields.io/badge/license-MIT-orange?style=for-the-badge" alt="License"/>
</p>

<h1 align="center">⚡ SoDEX Volume Bot</h1>
<p align="center">
  <i>Automated volume generation toolkit for the SoDEX DEX — grid trading on BTC-USD perps with maker-optimized execution.</i>
</p>

<p align="center">
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-features">Features</a> •
  <a href="#%EF%B8%8F-architecture">Architecture</a> •
  <a href="#-modes">Modes</a> •
  <a href="#-configuration">Configuration</a> •
  <a href="#-faq">FAQ</a>
</p>

---

## 📖 What Is This?

This bot automates volume generation on the **SoDEX** decentralized exchange. It places and cancels BTC-USD perpetual futures orders using **maker-only (GTX/Post-Only)** limit orders, keeping fees minimal while accumulating trade volume for airdrop eligibility.

> **TL;DR:** You deposit funds → the bot trades back and forth efficiently → you accumulate volume at near-zero cost.

---

## ✨ Features

### 🎯 Grid Trading Engine (`gridsodex.js`)
| Feature | Description |
|---------|-------------|
| **Adaptive Maker Orders** | Places GTX (Post-Only) limit orders at the best bid/ask to guarantee maker fees (0.012%) |
| **ATR-Based Volatility Regime** | Automatically adapts patience, sizing, and drift thresholds based on real-time 14-period ATR |
| **Loss-Aware Drift Escape** | If price drifts too far, switches to direct market close with balance-diff fill tracking |
| **WebSocket Price Feed** | Real-time bookTicker, markPrice, and candle streams via WS (REST fallback) |
| **Max Session Loss Protection** | Automatically stops if balance drops ≥ $2 to protect capital |
| **Orphan Position Cleanup** | Emergency market close for any stale positions at startup/shutdown |
| **Graceful Shutdown** | `Ctrl+C` safely cancels orders, closes positions, then exits |

### 🔄 Full Pipeline Orchestrator (`sodex.js`)
| Feature | Description |
|---------|-------------|
| **5 Operating Modes** | Deposit, Withdraw, ETH Arbitrage, Full Flow, Multi-Wallet Farm |
| **Multi-Wallet Support** | Sequential round-robin processing across unlimited wallets |
| **On-Chain Transfers** | ValueChain EVM-Funding ↔ Spot ↔ Futures automated transfers |
| **SOSO↔USDC Trading** | Smart maker buy/sell with automatic repricing |
| **Bybit Integration** | ETH arbitrage loop via Bybit API (deposit/withdraw/convert) |
| **EIP-712 Authentication** | Full typed-data signing, API key rotation, agent key management |

### 🛡️ Anti-Detection System
- Per-wallet browser profile consistency (UA + Sec-CH-UA + Platform)
- 13-header Chrome-identical request fingerprint via `wreq-js`
- Session warming (page visit before API calls)
- Human-like skewed delay distribution (not uniform random)

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────┐
│                   sodex.js                       │
│          (Orchestrator / CLI Menu)                │
│                                                  │
│  Mode 1: Deposit    SOSO → SoDEX → USDC → Perps │
│  Mode 2: Withdraw   Perps → USDC → SOSO → Wallet│
│  Mode 3: ETH Arb    ETH → SoDEX → Bybit → Next  │
│  Mode 4: Full Flow  Deposit → Trade → Withdraw   │
│  Mode 5: Farm       Multi-Wallet Sequential       │
│                                                  │
│  ┌──────────────────────────────────────┐        │
│  │           gridsodex.js               │        │
│  │      (Grid Trading Engine)           │        │
│  │                                      │        │
│  │  ┌───────────┐  ┌────────────────┐   │        │
│  │  │ WSManager │  │  SodexClient   │   │        │
│  │  │ (Realtime)│  │  (API + Sign)  │   │        │
│  │  └───────────┘  └────────────────┘   │        │
│  └──────────────────────────────────────┘        │
└──────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

- **Node.js** ≥ 18.0 ([Download](https://nodejs.org/))
- A funded wallet on [SoDEX](https://sodex.com/) (SOSO or USDC in Futures)
- Your wallet's private key

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/SDEX.git
cd SDEX
```

### 2️⃣ Install Dependencies

```bash
npm install
```

This installs:
- `ethers` — Ethereum wallet & EIP-712 signing
- `wreq-js` — Anti-detect HTTP client (Chrome TLS fingerprint)
- `ws` — WebSocket for real-time price feeds
- `dotenv` — Environment variable management
- `axios` — HTTP client (utility)

### 3️⃣ Set Up Your Wallet

Create a file called `PK.txt` in the project root:

```
# One private key per line (with or without 0x prefix)
0xYOUR_PRIVATE_KEY_HERE
```

> ⚠️ **IMPORTANT:** Never share or commit `PK.txt`. Add it to your `.gitignore`!

### 4️⃣ Configure Environment (Optional)

Only needed for **Mode 3 (ETH Arbitrage)**. Copy and fill in `.env`:

```env
# Bybit API (for ETH Arbitrage mode only)
BYBIT_API_KEY=your_key_here
BYBIT_API_SECRET=your_secret_here
BYBIT_BASE_URL=https://api.bybit.com

# Relay wallet for multi-wallet ETH forwarding
PENAMPUNG_PK=your_relay_wallet_pk

# Bybit SOSO deposit address (ValueChain)
BYBIT_SOSO_DEPOSIT=0x...
```

### 5️⃣ Run the Bot

**Grid Trading Bot (Standalone):**
```bash
node gridsodex.js
```

**Full Orchestrator (All Modes):**
```bash
node sodex.js
```

**CLI Shortcuts:**
```bash
node sodex.js deposit          # Deposit MAX SOSO
node sodex.js deposit 0.5      # Deposit 0.5 SOSO
node sodex.js withdraw         # Withdraw MAX SOSO
node sodex.js withdraw 100     # Withdraw 100 SOSO
```

---

## 📋 Modes

### Mode 1: Deposit
```
SOSO (ValueChain Wallet) → SoDEX Spot → Sell SOSO → Buy USDC → Transfer to Futures
```
Converts your SOSO tokens into USDC and deposits them into the Futures trading account.

### Mode 2: Withdraw
```
Futures → Spot (Transfer USDC) → Buy SOSO → Send to Next Wallet (ValueChain)
```
Extracts funds from Futures, converts back to SOSO, and sends to another wallet.

### Mode 3: ETH Arbitrage
```
ETH (Arbitrum) → SoDEX → Trade → SOSO → Bybit → Convert → ETH → Penampung → Next Wallet
```
Multi-wallet ETH recycling loop. Requires Bybit API keys.

### Mode 4: Full Flow
```
Deposit → Grid Trade (volume target) → Withdraw
```
Complete automated cycle: deposit, trade until target volume, then withdraw.

### Mode 5: Farm 🌾
```
Wallet 1 → Trade → Withdraw to Wallet 2 → Trade → ... → Wallet N
```
Sequential multi-wallet volume farming. Processes each wallet one at a time, passing funds to the next.

---

## ⚙️ Configuration

### Grid Bot Parameters

When you run the grid bot, it will prompt you for:

| Parameter | Description | Default |
|-----------|-------------|---------|
| **Target Volume** | USDC volume to generate (0 = infinite) | `0` (∞) |
| **Leverage** | Multiplier for position sizing (1-25x) | `20` |

### Volatility Regimes (Auto-Detected)

The bot automatically adapts based on BTC's 5-minute ATR:

| Regime | ATR Range | Patience | Drift Threshold | Size Multiplier |
|--------|-----------|----------|-----------------|-----------------|
| 🧘 **CALM** | < $50 | 25s | $3 | 1.2x |
| 📊 **NORMAL** | $50-$120 | 15s | $5 | 1.0x |
| ⚡ **VOLATILE** | > $120 | 9s | $4 | 0.8x |

### Fee Rates

| Type | Rate | Notes |
|------|------|-------|
| **Maker** | 0.012% | GTX Post-Only orders |
| **Taker** | 0.040% | Only used for drift escape |

### Cost Estimate

For every **$5,000** of generated volume:
- **Maker-only cost:** ~$0.60
- **With occasional taker escapes:** ~$0.80–$1.20

---

## 📁 Project Structure

```
SDEX/
├── gridsodex.js       # Grid trading engine (BTC-USD Perps)
├── sodex.js           # Main orchestrator (deposit/withdraw/farm)
├── package.json       # Dependencies and scripts
├── .env               # API keys & config (create this)
├── PK.txt             # Private keys (create this, DO NOT COMMIT)
├── agents.json        # Auto-generated API agent keys per wallet
└── README.md          # You are here!
```

---

## 🛑 Safety Features

| Feature | What It Does |
|---------|-------------|
| **Max Session Loss** | Stops trading if balance drops ≥ $2 |
| **Graceful Ctrl+C** | Cancels all orders → closes positions → exits cleanly |
| **Orphan Cleanup** | Detects and closes stale positions at startup |
| **Liquidation Detection** | WebSocket monitoring for liquidation events |
| **Failed Wallet Logging** | Saves failed wallet PKs to `pkwalletgagaldepwd.txt` for retry |

---

## ❓ FAQ

<details>
<summary><b>How much capital do I need?</b></summary>

Minimum **$5 USDC** in the Futures account. Recommended **$20–$50** for comfortable 20x leverage grid trading.
</details>

<details>
<summary><b>Is this safe? Can I lose money?</b></summary>

The bot uses Post-Only (maker) orders and closes positions within the same cycle. The primary cost is trading fees (~$0.60 per $5,000 volume). The max session loss protection stops the bot if you lose ≥ $2. However, in extreme market conditions (flash crashes), there's always some risk.
</details>

<details>
<summary><b>How fast does it generate volume?</b></summary>

Typically **$500–$2,000+ per hour** depending on market conditions and leverage. Calm markets with 20x leverage produce the highest throughput.
</details>

<details>
<summary><b>Can I run multiple wallets?</b></summary>

Yes! Add multiple private keys to `PK.txt` (one per line) and use **Mode 5 (Farm)**. The bot processes them sequentially, forwarding funds from wallet to wallet.
</details>

<details>
<summary><b>What if the bot crashes?</b></summary>

The bot includes retry logic and state detection on resume. For Mode 3 (ETH Arb), it checks Bybit and SoDEX balances to determine where to resume. For grid trading, any orphan positions are cleaned up at startup.
</details>

<details>
<summary><b>Do I need a VPS?</b></summary>

For short runs, your local machine is fine. For 24/7 farming, a VPS (e.g., Contabo, DigitalOcean) with `screen` or `pm2` is recommended.
</details>

---

## ⚠️ Disclaimer

This software is provided **as-is** for educational and research purposes. Trading cryptocurrency involves risk. The authors are not responsible for any financial losses. Use at your own discretion.

---

<p align="center">
  <b>Made with ☕ and 🧠 by the Aethereal Agenda team</b>
</p>
<p align="center">
  <i>If this tool helped you, consider dropping a ⭐ on the repo!</i>
</p>
