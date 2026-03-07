# Token Launch

Launch ERC-20 tokens on Base with a one-way bonding curve, progressive sell tax, and treasury-backed intrinsic value floor. Tokens are deployed via the OBSD LaunchPad smart contracts -- no external DEX liquidity required at launch.

## MCP Tools

| Tool | Description |
|------|-------------|
| `launch_token` | Deploy a new token with name, ticker, and creator wallet |
| `buy_token` | Buy tokens with ETH through the bonding curve |
| `sell_token` | Sell tokens at intrinsic value (IV) |
| `claim_fees` | Distribute pending creator fees as OBSD |
| `quote_buy` | Preview buy pricing without executing |
| `quote_sell` | Preview sell pricing without executing |
| `get_token_info` | Get token details, spot price, IV, supply |
| `list_launches` | List all tokens launched on the platform |
| `get_creator_earnings` | Check lifetime OBSD earned by a creator wallet |
| `get_platform_stats` | Platform-wide metrics |

## How It Works

1. **Deploy**: `launch_token("My Token", "MTK", "0xCreatorWallet")` creates an ERC-20 with a bonding curve
2. **Buy**: Users send ETH, receive tokens priced by the curve. 1% creator fee + 2% burn on each buy
3. **Sell**: Tokens sold at intrinsic value (Real ETH / Circulating Supply). Progressive sell tax from 25% (immediate) to 1% (30+ day hold)
4. **Earn**: Creators earn OBSD from every trade on their token -- 37.5% auto-distributed

## Key Properties

- **Intrinsic value only goes up** -- mathematically proven. Every trade increases IV for remaining holders
- **Zero token allocation to creator** -- no dump risk, creator earns OBSD only
- **No external liquidity needed** -- the contract is the market
- **Anti-rug by design** -- no owner functions, LP burned, supply only decreases
