# Configuration

Hound Flow analyzes the **Base** network (chain id `8453`). You can run on our shared keyless fallback or **bring your own key (BYOK)** for higher rate limits and reliability. All data-source settings live under **Data Source** in the dashboard.

## Data-source priority

Hound Flow uses the first configured source and falls back down the list if one fails:

| Priority | Source | What you provide | Where to get it |
|----------|--------|------------------|-----------------|
| 1 | **Basescan API key** | API key | https://basescan.org/myapikey |
| 2 | **Etherscan v2 (unified)** | API key | https://etherscan.io/myapikey — one key, all chains incl. Base via `chainid=8453` |
| 3 | **Custom RPC** | Base RPC URL | Your own node, Alchemy, Infura, QuickNode, etc. |
| 4 | **Keyless fallback** | nothing | Hound Flow's shared pool — rate-limited, best-effort (default) |

Each source plays a role:

- **Explorer keys (Basescan / Etherscan v2)** power verified-source lookups, ABIs, holder lists, and token transfers.
- **Custom RPC** powers low-level reads (`eth_call`, log queries) and honeypot simulation, with the lowest latency and no shared limits.

## Setting it up

1. Open **Data Source** in the dashboard.
2. Enter one or more of:
   - **Basescan API key**
   - **Etherscan v2 API key**
   - **Custom RPC URL** (`https://…`)
3. Click **Test Connection**. Hound Flow runs a read-only probe (`eth_blockNumber` for RPC, a sample query for an explorer key) and reports ✅ / ❌ per source.
4. Save. Credentials are **encrypted at rest** and shown only as a masked value (`••••last4`) afterward. They are write-only — no Hound Flow endpoint ever returns them.

> **Recommended setup:** a **custom Base RPC** for onchain reads **plus** a **Basescan or Etherscan v2 key** for verified-source, ABI, and holder data. With both, every tool has a first-class data path.

## How fallback works

If a configured source fails at call time, Hound Flow automatically tries the next available source. Each tool response notes which source answered (e.g. `source=basescan`, `source=etherscan`, `source=rpc`), so you can see when a fallback occurred.

## Notes & limits

- **BYOK keys are yours.** They serve only your account's tool calls and are never shared with other accounts.
- **Base only (MVP).** All configured sources must point at Base. Multi-chain is on the [roadmap](architecture.md#roadmap).
- **Rate limits.** The keyless fallback is rate-limited and best-effort. Add your own key or RPC to raise limits and improve reliability.

## Security

- API keys are stored as hashes and shown in plaintext only once at creation.
- BYOK credentials are encrypted at rest and decrypted only to serve your calls. They are never logged in plaintext or returned by any API.
- All traffic is over HTTPS. The MCP endpoint requires authentication — there is no anonymous access.

See [Getting Started](getting-started.md) to connect your agent, or the [MCP Tools reference](mcp-tools.md) for what each tool does.
