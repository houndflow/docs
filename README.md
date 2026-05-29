<div align="center">

# Hound Flow

**Give AI agents eyes onchain.**

Onchain intelligence infrastructure that lets AI agents investigate wallets, contracts, transactions, and suspicious activity on **Base** — autonomously, through a single MCP connection.

[Getting Started](docs/getting-started.md) · [Architecture](docs/architecture.md) · [MCP Tools](docs/mcp-tools.md) · [Configuration](docs/configuration.md)

</div>

---

## What is Hound Flow?

Blockchain data is public, but understanding it still takes manual block-explorer digging and years of analyst experience. Hound Flow closes that gap for AI agents.

Connect your agent (Cursor, Claude Code, Codex, or any MCP-compatible client) to our hosted MCP server with one API key, and it gains a seasoned onchain investigator's toolkit: rug-risk scoring, contract audits, wallet profiling, deployer tracing, transaction decoding, and holder analysis.

**Hound Flow is infrastructure, not a dashboard.** Where Etherscan, Arkham, and Nansen built UIs for humans, Hound Flow builds an MCP/API layer for autonomous agents.

## Why Hound Flow

- **Agent-native** — Investigation tools exposed over the Model Context Protocol, not a web UI.
- **One-key connect** — Whitelisted wallet logs in, generates a single API key, connects any MCP client.
- **Bring your own key (BYOK)** — Use your own Basescan / Etherscan v2 key or RPC endpoint, or run on our shared fallback.
- **Base-first** — Purpose-built for the Base ecosystem (chain id `8453`). Multi-chain on the roadmap.
- **Security-focused** — Read-only by design. No onchain writes, ever.

## Capabilities

| Tool | Investigates |
|------|--------------|
| `hound_detect_rug_risk` | Rug-pull risk: ownership, mint/freeze powers, LP lock, holder concentration |
| `hound_analyze_contract` | Verification, proxy/upgradeability, admin roles, mint/freeze/pause, backdoors |
| `hound_analyze_wallet` | Behavioral profile, funding source, counterparties, risk flags |
| `hound_check_deployer_history` | Every contract from a deployer; serial-rugger detection |
| `hound_decode_transaction` | Plain-English transaction decode + suspicious-approval flags |
| `hound_analyze_token_holders` | Holder distribution, concentration (HHI), LP/lock exclusions, whale clusters |

See the [full tool reference](docs/mcp-tools.md).

## Quick start

1. Connect your whitelisted wallet at **dashboard.houndflow.com** and generate your API key.
2. Add the MCP server to your agent:

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

3. Ask your agent: *"Use hound to scan rug risk for `0x…` on Base."*

Full walkthrough in [Getting Started](docs/getting-started.md).

## Documentation

| Doc | Contents |
|-----|----------|
| [Getting Started](docs/getting-started.md) | Access, API key, connecting your agent |
| [Architecture](docs/architecture.md) | System design and components |
| [Configuration](docs/configuration.md) | BYOK data sources (Basescan / Etherscan v2 / RPC) |
| [MCP Tools](docs/mcp-tools.md) | Tool-by-tool reference |

## Ecosystem

Hound Flow is designed to interoperate with the [Aeon](https://github.com/aaronjmars/aeon) autonomous-agent framework. Our investigation skills fill the security/forensics gap in Aeon's crypto skill set — monitoring tools answer *"what moved?"*; Hound Flow answers *"is this safe, and who is behind it?"*

## Status

Hound Flow is in active development (MVP). Access is wallet-gated during the early phase. Follow [@houndflow](https://github.com/houndflow) for updates.

## License

Documentation licensed under [CC BY 4.0](LICENSE). See [CONTRIBUTING](CONTRIBUTING.md) to propose changes.
