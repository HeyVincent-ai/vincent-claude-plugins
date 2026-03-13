# Security Scanner

Scan smart contracts on Base for common vulnerabilities before interacting with them. Detects honeypots, rug pull patterns, hidden mint functions, and suspicious bytecode.

## MCP Tools

| Tool | Description |
|------|-------------|
| `scan_contract` | Full security audit -- checks honeypot, ownership, mint, and bytecode patterns |
| `check_honeypot` | Quick check: can you actually sell this token? |
| `analyze_bytecode` | Low-level bytecode analysis for hidden functions |
| `check_ownership` | Who owns the contract? Can they pause, mint, or blacklist? |
| `get_risk_score` | Composite risk score from 0 (safe) to 100 (dangerous) |

## Common Workflows

### Before buying a token
1. `get_risk_score("0xTokenAddress")` -- get a quick safety check
2. If score > 50, run `scan_contract("0xTokenAddress")` for detailed findings
3. Check `check_honeypot("0xTokenAddress")` to verify you can sell

### Auditing a new contract
1. `scan_contract("0xContractAddress")` for the full report
2. `check_ownership("0xContractAddress")` to understand admin privileges
3. `analyze_bytecode("0xContractAddress")` for hidden functions not visible in source

## What It Detects

- **Honeypots**: Contracts that let you buy but prevent selling
- **Hidden mints**: Owner can inflate supply without limit
- **Blacklists**: Owner can freeze specific addresses
- **Pause functions**: Owner can halt all transfers
- **Proxy patterns**: Contract can be silently upgraded
- **Fee manipulation**: Owner can change buy/sell fees after deployment
