# OBSD LaunchPad Plugin for Claude Code

Three MCP servers giving Claude Code full Base chain DeFi capabilities: launch tokens, scan contracts for security issues, and manage wallets.

## Servers included

### obsd-launchpad-mcp (14 tools)

Token launch platform on Base with bonding curves, progressive sell tax, and treasury-backed intrinsic value floor.

| Tool | Description |
|------|-------------|
| `launch_token` | Deploy a new token with bonding curve |
| `buy_token` | Buy tokens with ETH |
| `sell_token` | Sell tokens for ETH |
| `claim_fees` | Distribute pending creator fees |
| `quote_buy` | Get a buy quote without executing |
| `quote_sell` | Get a sell quote without executing |
| `get_creator_earnings` | Check lifetime OBSD earned |
| `get_platform_stats` | Platform overview and metrics |
| `get_token_info` | Token details and pricing |
| `list_launches` | All launched tokens |
| `get_staking_info` | Staking vault details |
| `stake_obsd` | Stake OBSD to earn platform fees |
| `unstake_obsd` | Unstake OBSD |
| `get_referral_info` | Referral tracking data |

### base-security-scanner-mcp (5 tools)

Scan smart contracts on Base for vulnerabilities before interacting with them.

| Tool | Description |
|------|-------------|
| `scan_contract` | Full security scan -- honeypots, rug pulls, hidden mints |
| `check_honeypot` | Quick honeypot detection |
| `analyze_bytecode` | Bytecode-level vulnerability analysis |
| `check_ownership` | Ownership and admin function analysis |
| `get_risk_score` | Composite risk score (0-100) |

### base-wallet-toolkit-mcp (7 tools)

Multi-wallet management on Base -- balances, transfers, token tracking.

| Tool | Description |
|------|-------------|
| `get_balance` | ETH and token balances |
| `get_tokens` | List all tokens held |
| `transfer_eth` | Send ETH |
| `transfer_token` | Send ERC-20 tokens |
| `get_tx_history` | Transaction history |
| `get_gas_estimate` | Gas estimation |
| `watch_wallet` | Monitor wallet activity |

## Install

```
/plugin marketplace add lordbasilaiassistant-sudo/vincent-claude-plugins
/plugin install obsd-launchpad@obsd-launchpad
```

Or install individual servers via npx:

```bash
npx -y obsd-launchpad-mcp
npx -y base-security-scanner-mcp
npx -y base-wallet-toolkit-mcp
```

## Chain

All tools operate on **Base mainnet** (chainId 8453). Set `RPC_URL` environment variable to override the default RPC endpoint.

## Links

- [OBSD LaunchPad](https://github.com/lordbasilaiassistant-sudo/obsd-launchpad-mcp)
- [Security Scanner](https://github.com/lordbasilaiassistant-sudo/base-security-scanner-mcp)
- [Wallet Toolkit](https://github.com/lordbasilaiassistant-sudo/base-wallet-toolkit-mcp)
