# Vincent Wallet

Use the Vincent wallet MCP tools to transfer tokens, swap on DEXs, check balances, and interact with smart contracts on any EVM chain. The agent never sees the private key — all transactions are executed server-side through a ZeroDev smart account with policy enforcement.

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

## MCP Tools

| Tool | Description |
|---|---|
| `vincent_wallet_address` | Get the smart account address |
| `vincent_wallet_balances` | Get portfolio balances across all EVM chains |
| `vincent_wallet_transfer` | Transfer native ETH or ERC-20 tokens |
| `vincent_wallet_swap_preview` | Preview a token swap (pricing only, no execution) |
| `vincent_wallet_swap_execute` | Execute a token swap via 0x |
| `vincent_wallet_send_transaction` | Send an arbitrary EVM transaction (custom calldata) |

## Tool Parameters

### vincent_wallet_transfer
- `to` (string, required): Recipient address (0x...)
- `amount` (string, required): Human-readable amount (e.g. "0.1" for 0.1 ETH)
- `token` (string, optional): ERC-20 token contract address. Omit for native ETH.
- `chainId` (number, required): Chain ID (1=Ethereum, 137=Polygon, 42161=Arbitrum, 10=Optimism, 8453=Base)

### vincent_wallet_balances
- `chainIds` (number[], optional): Filter to specific chains. Omit for all chains.

### vincent_wallet_swap_preview / vincent_wallet_swap_execute
- `sellToken` (string, required): Token to sell. Use `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE` for native ETH.
- `buyToken` (string, required): Token to buy
- `sellAmount` (string, required): Human-readable amount to sell
- `chainId` (number, required): Chain ID
- `slippageBps` (number, optional): Slippage tolerance in basis points (100 = 1%). Default: 100. Execute only.

### vincent_wallet_send_transaction
- `to` (string, required): Contract address
- `data` (string, required): Hex-encoded calldata
- `value` (string, optional): Wei value to send
- `chainId` (number, required): Chain ID

## Common Workflows

### Check address and balances
1. Call `vincent_wallet_address` to get the smart account address
2. Call `vincent_wallet_balances` to see all token holdings with USD values

### Transfer tokens
1. Call `vincent_wallet_transfer` with `to`, `amount`, and optionally `token` and `chainId`
2. If the response has `status: "pending_approval"`, the wallet owner will be notified via Telegram

### Swap tokens
1. Call `vincent_wallet_swap_preview` to get pricing and expected output
2. Call `vincent_wallet_swap_execute` with the same parameters plus `slippageBps`
3. ERC-20 approvals are handled automatically

## Security

- **No gas needed.** All transaction gas fees are sponsored by a paymaster.
- **Private keys never leave the server.** The agent gets a scoped Bearer token, not a private key.
- **Server-side policy enforcement.** The wallet owner sets spending limits, address allowlists, token allowlists, function allowlists, and approval thresholds at [heyvincent.ai](https://heyvincent.ai). The agent cannot bypass policies.
- **Human-in-the-loop.** Transactions above the approval threshold are held and the wallet owner is notified via Telegram to approve or deny.
- If a transaction is rejected by a policy, the response explains which policy was triggered.

## Raw Signer (Advanced)

For raw ECDSA/Ed25519 signing when smart accounts won't work:

| Tool | Description |
|---|---|
| `vincent_raw_signer_addresses` | Get Ethereum (secp256k1) and Solana (ed25519) addresses |
| `vincent_raw_signer_sign` | Sign a hex-encoded message with `curve: "ethereum"` or `"solana"` |

## Important Notes

- Supported chains: Ethereum (1), Polygon (137), Arbitrum (42161), Optimism (10), Base (8453), and more
- Use `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE` as the token address for native ETH in swaps
- Balances include ERC-20 tokens and native balances with symbols, decimals, logos, and USD values
- The wallet owner manages policies and can revoke access at any time from [heyvincent.ai](https://heyvincent.ai)
