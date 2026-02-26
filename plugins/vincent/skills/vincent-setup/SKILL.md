# /vincent-setup

Set up your Vincent API key so the Vincent MCP server can authenticate.

## Instructions

When the user runs `/vincent-setup`, walk them through the following steps:

### Step 1: Check for existing key

Check if the environment variable `VINCENT_API_KEY` is already set:

```bash
echo $VINCENT_API_KEY
```

If it's set and starts with `ssk_`, let the user know they're already configured and ask if they want to verify the connection (skip to Step 4).

### Step 2: Get a Vincent account

If no key is set, tell the user:

> You need a Vincent API key to connect. Here's how to get one:
>
> 1. Go to **https://heyvincent.ai** and create an account
> 2. Create a new account (Smart Wallet, Polymarket Wallet, or Data Source)
> 3. Go to the **API Keys** tab and create a new key
> 4. Copy the key (starts with `ssk_`) -- it's only shown once

### Step 3: Set the environment variable

Tell the user to export the key in their shell profile so it persists across sessions:

```bash
# Add to your ~/.zshrc or ~/.bashrc
export VINCENT_API_KEY="ssk_your_key_here"
```

Then reload their shell or run `source ~/.zshrc`.

### Step 4: Verify the connection

After the key is set, confirm the Vincent MCP server is available by checking if Vincent tools appear in the current session. If they do, congratulate the user and suggest example prompts based on their account type:

**Wallet prompts:**
- "Show my Vincent wallet address and balances"
- "Transfer 0.01 ETH to 0x..."
- "Swap 10 USDC for ETH on Base"

**Polymarket prompts:**
- "Find active prediction markets about the 2026 election"
- "Show my Polymarket positions and P&L"
- "Place a $25 bet on YES for the top market"

**Data source prompts:**
- "Search the web for the latest crypto news"
- "Find recent tweets about Solana"

If the tools don't appear, suggest restarting Claude Code to reload the MCP server.
