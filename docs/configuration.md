# Configuration

Hound Flow analyzes the **Base** network (chain id `8453`). You can run on our shared keyless fallback or **bring your own key (BYOK)** for higher rate limits and reliability.

## Client-side BYOK

**Hound Flow never stores your data-source secrets.** Your Basescan / Etherscan v2 key or RPC URL stays with your client — you supply it where you configure the MCP connection, it's used in-memory to serve your call, and it is never written to our database.

There are two ways to run, depending on how strict you want to be:

| Mode | How keys travel | Use when |
|------|-----------------|----------|
| **Hosted** (`mcp.houndflow.com`) | Sent as request headers over TLS, used in-memory for the single upstream call, **never stored** | Easiest setup; you're fine with keys passing through our server transiently |
| **Local** (`npx @houndflow/mcp`) | Read from your own environment; the server runs on your machine and calls Basescan / RPC **directly** | You want keys to **never leave your machine** (zero-transit) |

> **Transparency note:** In **hosted** mode your BYOK secrets do pass *through* our server to reach Basescan/RPC. They are transient and never stored — but if you need keys to never touch our infrastructure at all, run **local** mode.

## Data-source priority

In either mode, Hound Flow uses the first source you provide and falls back down the list:

| Priority | Source | What you provide | Where to get it |
|----------|--------|------------------|-----------------|
| 1 | **Basescan API key** | API key | https://basescan.org/myapikey |
| 2 | **Etherscan v2 (unified)** | API key | https://etherscan.io/myapikey — one key, all chains incl. Base via `chainid=8453` |
| 3 | **Custom RPC** | Base RPC URL | Your node, Alchemy, Infura, QuickNode, etc. |
| 4 | **Keyless fallback** | nothing | Hound Flow's shared pool — rate-limited, best-effort (default) |

Each response notes which source answered (`source=basescan|etherscan|rpc`).

## Hosted mode — set keys via headers

Add your BYOK credentials as headers alongside your Hound API key in your MCP client config. All are optional; omit them to use the keyless fallback.

```json
{
  "mcpServers": {
    "hound": {
      "url": "https://mcp.houndflow.com/mcp",
      "headers": {
        "Authorization": "Bearer hf_live_YOUR_KEY",
        "X-Basescan-Key": "YOUR_BASESCAN_KEY",
        "X-Base-Rpc-Url": "https://your-base-rpc"
      }
    }
  }
}
```

Supported headers: `X-Basescan-Key`, `X-Etherscan-Key`, `X-Base-Rpc-Url`.

## Local mode — set keys via environment

Run the server on your own machine; keys come from your environment and never transit our infrastructure.

```json
{
  "mcpServers": {
    "hound": {
      "command": "npx",
      "args": ["-y", "@houndflow/mcp"],
      "env": {
        "HOUND_API_KEY": "hf_live_YOUR_KEY",
        "BASESCAN_API_KEY": "YOUR_BASESCAN_KEY",
        "BASE_RPC_URL": "https://your-base-rpc"
      }
    }
  }
}
```

Supported env vars: `BASESCAN_API_KEY`, `ETHERSCAN_API_KEY`, `BASE_RPC_URL`. (`HOUND_API_KEY` still identifies your account.)

## Notes & limits

- **Your keys are yours.** They serve only your own calls and are never shared or stored.
- **Base only (MVP).** All sources must point at Base. Multi-chain is on the [roadmap](architecture.md#roadmap).
- **Rate limits.** The keyless fallback is rate-limited and best-effort. Add your own key or RPC to raise limits.
- If a source fails at call time, Hound Flow tries the next available one.

## Security

- Your **Hound API key** is stored only as a hash and shown in plaintext once at creation.
- Your **BYOK secrets are never stored** by Hound Flow — not in plaintext, not encrypted, not at all. In hosted mode they're held in memory only for the duration of a single call and redacted from logs; in local mode they never leave your machine.
- All traffic is over HTTPS. The hosted MCP endpoint requires authentication — there is no anonymous access.

See [Getting Started](getting-started.md) to connect your agent, or the [MCP Tools reference](mcp-tools.md) for what each tool does.
