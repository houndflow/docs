<div align="center">

# Hound Flow

**Give AI agents eyes onchain.**

Onchain investigation skills for AI agents on **Base** — rug-risk scoring, contract audits, wallet profiling, deployer tracing, transaction decoding, and holder analysis.

[Getting Started](docs/getting-started.md) · [Skills](docs/mcp-tools.md) · [Configuration](docs/configuration.md) · [Architecture](docs/architecture.md)

</div>

---

## What is Hound Flow?

Blockchain data is public, but understanding it still takes manual block-explorer digging and years of analyst experience. Hound Flow closes that gap for AI agents.

Hound Flow ships a set of **onchain investigation skills**. Where most crypto agent tooling answers *"what moved?"*, Hound Flow answers *"is this safe, and who is behind it?"* — a seasoned investigator's toolkit, packaged for autonomous agents.

## Available now: the Hound skill pack for Aeon

## Available now: merged into Aeon

Hound Flow's onchain-investigation skills are **merged into the [Aeon](https://github.com/aaronjmars/aeon) autonomous-agent framework** (PR #269). Install any of them into your own Aeon agent — they run on public Base endpoints out of the box, or with your own key (BYOK) for higher limits — no platform account required.

```bash
# install a Hound skill into your Aeon agent
./add-skill aaronjmars/aeon rug-scan

# or several
./add-skill aaronjmars/aeon rug-scan contract-audit wallet-profile
```

## The Hound Flow platform

A hosted platform complements the skill pack for users who want investigation as a managed service:

- **`chat.houndflow.com`** — **Hound Agent**, an OSINT chat: sign in with a whitelisted wallet and investigate onchain in natural language. The Hound tools are auto-connected, fund-flow graphs render inline, and you can bring your own Basescan key / RPC. *(Live.)*
- **`mcp.houndflow.com`** — a hosted MCP server exposing the Hound tools to any MCP client (Cursor, Claude Code, Codex), authenticated with one API key. *(Live.)*
- **`houndflow.com/dashboard`** — a profile dashboard: connect a whitelisted wallet (SIWE) and generate, rotate, or revoke your API key. *(Live.)*

Use Hound Flow via the chat app, the hosted MCP server, or by installing the skill pack into your own Aeon agent.

## Skills

| Skill | Investigates |
|-------|--------------|
| `rug-scan` | Rug-pull risk: ownership, mint/freeze powers, LP lock, holder concentration |
| `contract-audit` | Verification, proxy/upgradeability, admin roles, mint/freeze/pause, backdoors |
| `wallet-profile` | Behavioral profile, funding source, counterparties, risk flags |
| `deployer-trace` | Every contract from a deployer; serial-rugger detection |
| `tx-explain` | Plain-English transaction decode + suspicious-approval flags |
| `holder-concentration` | Holder distribution, concentration (HHI), LP/lock exclusions, whale clusters |
| `fund-flow` | Trace where funds move across hops; auto-generated flow graph |

See the [full skill reference](docs/mcp-tools.md).

## Why Hound Flow

- **Investigation-first** — Security and forensics, not just price/volume monitoring.
- **Base-native** — Purpose-built for the Base ecosystem (chain id `8453`). Multi-chain on the roadmap.
- **Bring your own key (BYOK)** — Use your own Basescan / Etherscan v2 key or RPC endpoint, or run on public endpoints.
- **Read-only by design** — Tools never sign or send transactions. No onchain writes, ever.
- **Standalone** — Skills work in any Aeon agent today; the hosted platform is optional and additive.

## Documentation

| Doc | Contents |
|-----|----------|
| [Getting Started](docs/getting-started.md) | Use the chat app · connect the hosted MCP · install skills into Aeon |
| [Skills](docs/mcp-tools.md) | Skill-by-skill reference |
| [Configuration](docs/configuration.md) | BYOK data sources (Basescan / Etherscan v2 / RPC) |
| [Architecture](docs/architecture.md) | How skills and the planned platform fit together |

## Status

The core platform is live: the **chat app** (chat.houndflow.com), the **hosted MCP server** (mcp.houndflow.com), and the account **dashboard** are all running, and the six investigation skills are **merged into Aeon** ([#269](https://github.com/aaronjmars/aeon/pull/269)). Multi-chain support and additional skills are on the roadmap. Follow [@houndflow](https://github.com/houndflow) for updates.

## License

Documentation licensed under [CC BY 4.0](LICENSE). See [CONTRIBUTING](CONTRIBUTING.md) to propose changes.
