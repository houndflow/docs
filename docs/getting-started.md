# Getting Started

This guide takes you from zero to your first onchain investigation in a few minutes.

## Prerequisites

- A **whitelisted wallet** (Hound Flow is wallet-gated during MVP).
- An **MCP-compatible AI client** — Cursor, Claude Code, Codex, or any client that speaks MCP over Streamable HTTP.

## 1. Sign in

1. Visit **https://dashboard.houndflow.com**.
2. Click **Connect Wallet** and sign the login message (SIWE — Sign-In With Ethereum). This proves wallet ownership; it costs no gas and sends no transaction.
3. If your wallet is whitelisted, you enter the dashboard. Otherwise you'll see a **Request Access** screen that adds you to the waitlist.

> One wallet = one account. There is no password — login is the wallet signature.

## 2. Generate your API key

Each account has **exactly one** API key.

1. Open **API Key** in the dashboard.
2. Click **Generate Key**. The key is shown **once**:
   ```
   hf_live_3f9a2c7b8e1d4a6f0c5b9e2d7a1f4c8b
   ```
3. Copy and store it securely (password manager or your agent's secret store). Hound Flow stores only a hash and cannot show it again.
4. To replace a lost or leaked key, use **Rotate** (revokes the old key, issues a new one). **Revoke** disables access entirely.

## 3. (Optional) Configure a data source

Hound Flow works out of the box on a shared, rate-limited fallback. For higher limits and reliability, add your own Basescan / Etherscan v2 key or RPC endpoint under **Data Source**. See [Configuration](configuration.md) for details.

## 4. Connect your agent

Hound Flow's MCP server lives at **`https://mcp.houndflow.com/mcp`** (Streamable HTTP). Authenticate with your API key as a Bearer token. Replace `hf_live_YOUR_KEY` below.

### Cursor — `~/.cursor/mcp.json`
```json
{
  "mcpServers": {
    "hound": {
      "url": "https://mcp.houndflow.com/mcp",
      "headers": { "Authorization": "Bearer hf_live_YOUR_KEY" }
    }
  }
}
```

### Claude Code — `.mcp.json` (project root)
```json
{
  "mcpServers": {
    "hound": {
      "type": "http",
      "url": "https://mcp.houndflow.com/mcp",
      "headers": { "Authorization": "Bearer hf_live_YOUR_KEY" }
    }
  }
}
```

Or via CLI:
```bash
claude mcp add --transport http hound https://mcp.houndflow.com/mcp \
  --header "Authorization: Bearer hf_live_YOUR_KEY"
```

### Codex / other MCP clients
Point the client at the HTTP MCP URL with the same `Authorization: Bearer` header. Any client speaking MCP over Streamable HTTP works.

> The dashboard's **Connect** tab generates these snippets with your key already filled in.

## 5. Run your first investigation

In your agent, try:

```
Use hound to scan rug risk for 0x4200000000000000000000000000000000000006 on Base
```

You should receive a structured rug-risk verdict. Explore the other tools:

- *"Audit contract 0x… on Base"* → `hound_analyze_contract`
- *"Profile wallet 0x…"* → `hound_analyze_wallet`
- *"What else did 0x… deploy?"* → `hound_check_deployer_history`
- *"Explain transaction 0x…"* → `hound_decode_transaction`
- *"Show holder concentration for 0x…"* → `hound_analyze_token_holders`

See the [MCP Tools reference](mcp-tools.md) for inputs and outputs.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `401 unauthorized` | Missing / invalid / revoked key | Check the Bearer header; rotate the key in the dashboard |
| `403 not_whitelisted` | Wallet not approved | Request access and wait for approval |
| `409 key_exists` on generate | Account already has a key | Use **Rotate** instead |
| `DATA_SOURCE_ERROR` from a tool | All configured sources failed | Set or repair a data source; run **Test Connection** |
| Rate-limited | Shared keyless limits | Add your own BYOK key / RPC |

## Next steps

- [Configuration](configuration.md) — bring your own data source
- [MCP Tools](mcp-tools.md) — full tool reference
- [Architecture](architecture.md) — how it all fits together
