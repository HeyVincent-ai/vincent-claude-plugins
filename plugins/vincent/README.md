# Vincent Plugin for Claude Code

Give Claude Code access to secure crypto wallets, Polymarket trading, and real-time data through [Vincent](https://heyvincent.ai).

All tools work via MCP — no CLI or Node.js required. Authentication is handled automatically through the `VINCENT_API_KEY` environment variable.

## What you get

Once installed, Claude Code gains access to these tools (based on your account type):

| Account Type | Tools |
|---|---|
| **Smart Wallet** | Transfer tokens, swap tokens, check balances, get wallet address |
| **Polymarket Wallet** | Browse markets, place bets, view positions, check balance, cancel orders |
| **Data Sources** | Web search, news search, tweet search, user profiles |
| **EOA Signer** | Sign messages, get signing addresses |

All actions go through Vincent's policy engine with audit logging. Your private keys never leave Vincent's secure infrastructure.

## Included Skills

This plugin ships with the following skills:

| Skill | Description |
|---|---|
| **wallet** | Transfer tokens, swap on DEXs, check balances, and interact with smart contracts on any EVM chain. |
| **polymarket** | Browse prediction markets, place bets, view positions, check balances, and cancel orders on Polymarket. |
| **trading-engine** | Automated trading strategies with instruments, thesis, drivers, escalation policies, signal pipelines, LLM-powered decisions, and full analytics (P&L, signal/decision/trade logs). Standalone stop-loss, take-profit, and trailing stop rules. Emergency kill switch. |
| **brave-search** | Real-time web and news search via Brave Search with pay-per-call billing. |
| **twitter** | Search tweets, look up user profiles, and get recent activity from Twitter/X. |
| **credentials** | Securely store and manage credentials (API keys, passwords, tokens). CLI-based — manage via the [Vincent dashboard](https://heyvincent.ai) in the plugin context. |
| **vincent-setup** | Guided setup wizard for configuring your Vincent API key and verifying the connection. Run `/vincent-setup` to get started. |

## Setup

### 1. Install the plugin

```bash
/plugin marketplace add HeyVincent-ai/vincent-claude-plugins
/plugin install vincent@vincent-plugins
```

### 2. Get your API key

1. Go to [heyvincent.ai](https://heyvincent.ai) and create an account
2. Create a Smart Wallet, Polymarket Wallet, or Data Source
3. Go to the **API Keys** tab and create a new key
4. Copy the key (starts with `ssk_`)

### 3. Set your API key

```bash
export VINCENT_API_KEY="ssk_your_key_here"
```

Add this to your `~/.zshrc` or `~/.bashrc` to persist across sessions.

### 4. Restart Claude Code

Restart Claude Code to load the Vincent MCP server. You should see Vincent tools available.

Run `/vincent-setup` if you need help with the setup process.

## Security

- Your private keys and secrets never leave Vincent's infrastructure
- API keys use one-way SHA-256 hashing
- Policy controls let you set spending limits, allowlists, and approval requirements
- Full audit trail for every action
- Optional Telegram-based human-in-the-loop approval

## Links

- [Vincent Dashboard](https://heyvincent.ai)
- [Documentation](https://github.com/HeyVincent-ai/Vincent/tree/main/docs)
- [GitHub](https://github.com/HeyVincent-ai/Vincent)
