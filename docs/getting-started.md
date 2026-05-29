# Getting Started

There are three ways to use Hound Flow — all live today:

- **Path A — Chat app (easiest).** Investigate onchain in natural language at `chat.houndflow.com`.
- **Path B — Hosted MCP server.** Connect any MCP client (Cursor, Claude Code, Codex) with one API key.
- **Path C — Aeon skills.** Install the Hound skills into your own [Aeon](https://github.com/aaronjmars/aeon) agent.

All require a **whitelisted wallet** during early access.

---

## Path A — Chat app (now)

1. Go to **https://chat.houndflow.com**.
2. **Connect your wallet** (you'll be prompted to switch to Base) and sign in (SIWE — an off-chain signature, no gas). Non-whitelisted wallets are pointed to the waitlist.
3. Chat with **Hound Agent** — the investigation tools are auto-connected. Ask things like *"scan rug risk for 0x…"* or *"trace the fund flow from 0x…"* and fund-flow graphs render inline.
4. Optional: open **BYOK** in the menu to add your own Basescan key or RPC URL for faster, unthrottled data.

---

## Path B — Hosted MCP server (now)

The hosted MCP server at **`mcp.houndflow.com`** exposes the Hound tools to any MCP client with a single API key.

1. Get your API key at **houndflow.com/dashboard** — connect a whitelisted wallet (SIWE), then generate your key (shown once; only a hash is stored — Rotate/Revoke anytime).
2. Add the server to your MCP client:

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

To bring your own data source, add `X-Basescan-Key` / `X-Base-Rpc-Url` headers — see [Configuration](configuration.md).

---

## Path C — Install Hound skills into Aeon (now)

The six investigation skills are merged into Aeon ([#269](https://github.com/aaronjmars/aeon/pull/269)). From your Aeon repo root:

```bash
# install one skill
./add-skill aaronjmars/aeon rug-scan

# install several
./add-skill aaronjmars/aeon rug-scan contract-audit wallet-profile
```

Installed skills land in `skills/` and are added to `aeon.yml` **disabled**. Enable the ones you want:

```yaml
# aeon.yml
skills:
  rug-scan:
    enabled: true
```

Skills use public Base endpoints by default; to raise limits set any of these in your Aeon environment (see [Configuration](configuration.md)):

```bash
BASESCAN_API_KEY=...      # https://basescan.org/myapikey
ETHERSCAN_API_KEY=...     # https://etherscan.io/myapikey (Base via chainid=8453)
BASE_RPC_URL=...          # your own Base RPC endpoint
```

---

## Next steps

- [Skills reference](mcp-tools.md) — what each skill investigates
- [Configuration](configuration.md) — bring your own data source
- [Architecture](architecture.md) — how the chat, MCP server, and skills fit together
