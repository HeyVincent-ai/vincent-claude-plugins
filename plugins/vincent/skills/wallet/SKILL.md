# Vincent Wallet

Use the Vincent wallet MCP tools to transfer tokens, swap on DEXs, check balances, and interact with smart contracts on any EVM chain. The agent never sees the private key — all transactions are executed server-side through a ZeroDev smart account with policy enforcement.

**Vincent is open source and audited.** The server-side code that enforces policies, manages private keys, and executes transactions is publicly auditable at [github.com/HeyVincent-ai/Vincent](https://github.com/HeyVincent-ai/Vincent).

Authentication is handled automatically by the MCP server via `VINCENT_API_KEY`.

## Which Secret Type to Use

| Type         | Use Case                                  | Network                 | Gas              |
| ------------ | ----------------------------------------- | ----------------------- | ---------------- |
| `EVM_WALLET` | Transfers, swaps, DeFi, contract calls    | Any EVM chain           | Sponsored (free) |
| `RAW_SIGNER` | Raw message signing for special protocols | Any (Ethereum + Solana) | You pay          |

**Choose `EVM_WALLET`** (default) for sending ETH/tokens, swapping, and interacting with smart contracts.

**Choose `RAW_SIGNER`** only when you need raw ECDSA/Ed25519 signatures for protocols that don't work with smart accounts, or for Solana signatures.

## MCP Tools

| Tool | Description |
|---|---|
| `vincent_wallet_address` | Get the smart account address |
| `vincent_wallet_balances` | Get portfolio balances across all EVM chains |
| `vincent_wallet_transfer` | Transfer native ETH or ERC-20 tokens |
| `vincent_wallet_swap_preview` | Preview a token swap (pricing only, no execution) |
| `vincent_wallet_swap_execute` | Execute a token swap via 0x |
| `vincent_wallet_send_transaction` | Send an arbitrary EVM transaction (custom calldata) |
| `vincent_wallet_transfer_between_preview` | Preview a transfer between your Vincent secrets (get quote) |
| `vincent_wallet_transfer_between_execute` | Execute a transfer between your Vincent secrets |
| `vincent_wallet_transfer_between_status` | Check status of a cross-chain transfer between secrets |

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

### vincent_wallet_transfer_between_preview / vincent_wallet_transfer_between_execute
- `toSecretId` (string, required): Destination secret ID (EVM_WALLET or POLYMARKET_WALLET)
- `fromChain` (number, required): Source chain ID
- `toChain` (number, required): Destination chain ID
- `tokenIn` (string, required): Token address on source chain (or `ETH`)
- `amount` (number, required): Amount to transfer
- `tokenOut` (string, required): Token address on destination chain (or `ETH`)
- `slippage` (number, optional): Slippage in basis points (execute only)

### vincent_wallet_transfer_between_status
- `relayId` (string, required): Relay request ID (from cross-chain transfer response)

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

### Transfer between your secrets
Transfer funds between Vincent secrets you own (e.g., from one EVM wallet to another, or to a Polymarket wallet):
1. Call `vincent_wallet_transfer_between_preview` to get a quote
2. Call `vincent_wallet_transfer_between_execute` to execute
3. For cross-chain transfers, use `vincent_wallet_transfer_between_status` to check progress

**Behavior:**
- **Same token + same chain**: Executes as a direct transfer (gas sponsored)
- **Different token or chain**: Uses a relay service for atomic swap + bridge
- The destination secret can be an `EVM_WALLET` or `POLYMARKET_WALLET`
- The server verifies you own both secrets — transfers to secrets you don't own are rejected

## Output Format

Balance response:

```json
{
  "address": "0x...",
  "balances": [
    {
      "token": "ETH",
      "balance": "0.5",
      "usdValue": "1250.00"
    }
  ]
}
```

Transaction response:

```json
{
  "transactionHash": "0x...",
  "status": "confirmed"
}
```

For transactions requiring human approval:

```json
{
  "status": "pending_approval",
  "message": "Transaction requires owner approval via Telegram"
}
```

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| `401 Unauthorized` | Invalid or missing API key | Check that `VINCENT_API_KEY` is set correctly |
| `403 Policy Violation` | Transaction blocked by server-side policy | User must adjust policies at heyvincent.ai |
| `400 Insufficient Balance` | Not enough tokens for the transfer | Check balances before transferring |
| `429 Rate Limited` | Too many requests | Wait and retry with backoff |
| `pending_approval` | Transaction exceeds approval threshold | User will receive Telegram notification to approve/deny |

## Policies (Server-Side Enforcement)

The wallet owner controls what the agent can do by setting policies at [heyvincent.ai](https://heyvincent.ai). All policies are enforced server-side — the agent cannot bypass or modify them. The policy enforcement logic is open source and auditable at [github.com/HeyVincent-ai/Vincent](https://github.com/HeyVincent-ai/Vincent).

| Policy                      | What it does                                                        |
| --------------------------- | ------------------------------------------------------------------- |
| **Address allowlist**       | Only allow transfers/calls to specific addresses                    |
| **Token allowlist**         | Only allow transfers of specific ERC-20 tokens                      |
| **Function allowlist**      | Only allow calling specific contract functions (by 4-byte selector) |
| **Spending limit (per tx)** | Max USD value per transaction                                       |
| **Spending limit (daily)**  | Max USD value per rolling 24 hours                                  |
| **Spending limit (weekly)** | Max USD value per rolling 7 days                                    |
| **Require approval**        | Every transaction needs human approval via Telegram                 |
| **Approval threshold**      | Transactions above a USD amount need human approval via Telegram    |

Before the wallet is claimed, the agent can operate without policy restrictions. Once the human operator claims the wallet, they can add policies to constrain the agent's behavior and can revoke the agent's API key entirely at any time.

## Raw Signer (Advanced)

For raw ECDSA/Ed25519 signing when smart accounts won't work:

| Tool | Description |
|---|---|
| `vincent_raw_signer_addresses` | Get Ethereum (secp256k1) and Solana (ed25519) addresses |
| `vincent_raw_signer_sign` | Sign a hex-encoded message with `curve: "ethereum"` or `"solana"` |

## Important Notes

- **No gas needed.** A paymaster is fully set up — all transaction gas fees are sponsored automatically.
- **Never try to access raw secret values.** The private key stays server-side — that's the whole point.
- Supported chains: Ethereum (1), Polygon (137), Arbitrum (42161), Optimism (10), Base (8453), and more.
- Use `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE` as the token address for native ETH in swaps.
- Balances include ERC-20 tokens and native balances with symbols, decimals, logos, and USD values.
- If a transaction is rejected, it may be blocked by a server-side policy. Tell the user to check their policy settings at [heyvincent.ai](https://heyvincent.ai).
- If a transaction requires approval, it will return `status: "pending_approval"`. The wallet owner will receive a Telegram notification to approve or deny.
- The wallet owner manages policies and can revoke access at any time from [heyvincent.ai](https://heyvincent.ai).
