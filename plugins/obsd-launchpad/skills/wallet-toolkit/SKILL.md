# Wallet Toolkit

Manage wallets on Base -- check balances, transfer tokens, and monitor activity. Supports ETH and any ERC-20 token on Base mainnet.

## MCP Tools

| Tool | Description |
|------|-------------|
| `get_balance` | Get ETH and token balances for any address |
| `get_tokens` | List all ERC-20 tokens held by an address |
| `transfer_eth` | Send ETH to an address |
| `transfer_token` | Send ERC-20 tokens to an address |
| `get_tx_history` | Get recent transaction history |
| `get_gas_estimate` | Estimate gas cost for a transaction |
| `watch_wallet` | Monitor an address for incoming/outgoing transactions |

## Common Workflows

### Check portfolio
1. `get_balance("0xYourAddress")` -- see ETH balance
2. `get_tokens("0xYourAddress")` -- see all token holdings with values

### Send tokens
1. `get_gas_estimate(...)` -- check current gas costs
2. `transfer_eth("0xRecipient", "0.1")` or `transfer_token("0xToken", "0xRecipient", "100")`

### Monitor activity
1. `watch_wallet("0xAddress")` -- get notified of new transactions
2. `get_tx_history("0xAddress")` -- review recent activity

## Chain

All operations target **Base mainnet** (chainId 8453) by default. Gas costs on Base are typically under 0.000005 ETH per transaction.
