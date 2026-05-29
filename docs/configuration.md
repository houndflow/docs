# Configuration

Hound Flow analyzes the **Base** network (chain id `8453`). Skills run on public Base endpoints out of the box, or with **your own key (BYOK)** for higher rate limits and reliability.

## Data-source priority

Hound Flow uses the first source you provide and falls back down the list:

| Priority | Source | What you provide | Where to get it |
|----------|--------|------------------|-----------------|
| 1 | **Basescan API key** | API key | https://basescan.org/myapikey |
| 2 | **Etherscan v2 (unified)** | API key | https://etherscan.io/myapikey — one key, all chains incl. Base via `chainid=8453` |
| 3 | **Custom RPC** | Base RPC URL | Your node, Alchemy, Infura, QuickNode, etc. |
| 4 | **Public fallback** | nothing | Public Base endpoints — rate-limited, best-effort (default) |

Each response notes which source answered (`source=basescan|etherscan|rpc`).

## Skill pack (now) — set keys via environment

When running Hound skills inside an Aeon agent, provide your data source through the agent's environment. All are optional; omit them to use public endpoints.

```bash
BASESCAN_API_KEY=...      # https://basescan.org/myapikey
ETHERSCAN_API_KEY=...     # https://etherscan.io/myapikey (Base via chainid=8453)
BASE_RPC_URL=...          # your own Base RPC endpoint
```

> **Recommended combo:** a **custom Base RPC** (for onchain reads) **plus** a **Basescan or Etherscan v2 key** (for verified-source, ABI, and holder data). With both, every skill has a first-class data path.

Your keys stay in your own Aeon environment — Hound Flow has no server in this path.

## Hosted MCP (coming soon) — client-side BYOK

When the hosted MCP server launches, you'll pass BYOK credentials from your MCP client. **Hound Flow will never store them.** Two modes:

| Mode | How keys travel | Use when |
|------|-----------------|----------|
| **Hosted** (`mcp.houndflow.com`) | Sent as request headers over TLS, used in-memory for the single call, **never stored** | Easiest setup; fine with keys passing through transiently |
| **Local** (`npx @houndflow/mcp`) | Read from your own environment; runs on your machine, calls Base **directly** | You want keys to **never leave your machine** (zero-transit) |

Hosted-mode headers (planned): `X-Basescan-Key`, `X-Etherscan-Key`, `X-Base-Rpc-Url`, alongside `Authorization: Bearer hf_live_…`.

> **Transparency note:** in hosted mode BYOK secrets pass *through* the server to reach Basescan/RPC — transient, never stored. For zero-transit, run local mode.

## Notes & limits

- **Your keys are yours.** They serve only your own calls and are never shared or stored.
- **Base only (MVP).** All sources must point at Base. Multi-chain is on the [roadmap](architecture.md#roadmap).
- **Rate limits.** The public fallback is rate-limited and best-effort. Add your own key or RPC to raise limits.
- If a source fails at call time, Hound Flow tries the next available one.

## Security

- **BYOK secrets are never stored by Hound Flow.** In the skill pack they live in your own Aeon environment; in the planned hosted mode they're held in memory for a single call and redacted from logs; in local mode they never leave your machine.
- When the hosted platform launches, your Hound API key will be stored only as a hash and shown in plaintext once at creation.
- All traffic is over HTTPS. The hosted MCP endpoint will require authentication — no anonymous access.

See [Getting Started](getting-started.md) to install the skill pack, or the [Skills reference](mcp-tools.md) for what each skill does.
