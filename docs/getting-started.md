# Getting Started

Hound Flow's investigation skills run today inside the [Aeon](https://github.com/aaronjmars/aeon) agent framework. A hosted MCP server is coming for users who'd rather connect a managed endpoint.

- **Path A — Install the Hound skill pack (available now).** Run Hound's skills in your own Aeon agent.
- **Path B — Connect to the hosted MCP server (coming soon).** One API key, any MCP client.

---

## Path A — Install the Hound skill pack (now)

### Prerequisites

- An [Aeon](https://github.com/aaronjmars/aeon) agent (fork the repo, or your own clone).
- Optional: a Basescan / Etherscan v2 API key or a Base RPC URL for higher rate limits. Skills run on public endpoints without one. See [Configuration](configuration.md).

### Install

From your Aeon repo root:

```bash
# install one skill
./add-skill houndflow/hound-skills rug-scan

# install several
./add-skill houndflow/hound-skills rug-scan contract-audit wallet-profile

# install the whole pack
./add-skill houndflow/hound-skills --all
```

Installed skills land in `skills/` and are added to `aeon.yml` **disabled**. Enable the ones you want:

```yaml
# aeon.yml
skills:
  rug-scan:
    enabled: true
```

### (Optional) Add your data source

Skills use public Base endpoints by default. To raise limits, set any of these in your Aeon environment:

```bash
BASESCAN_API_KEY=...      # https://basescan.org/myapikey
ETHERSCAN_API_KEY=...     # https://etherscan.io/myapikey (works for Base via chainid=8453)
BASE_RPC_URL=...          # your own Base RPC endpoint
```

See [Configuration](configuration.md) for the resolution order and details.

### Run

Trigger a skill on demand or let it run on its schedule. For example, `rug-scan` takes a token address as its variable input and returns a structured rug-risk verdict. See the [Skills reference](mcp-tools.md) for each skill's input and output.

---

## Path B — Hosted MCP server (coming soon)

A hosted MCP server at **`mcp.houndflow.com`** will expose the same tools to any MCP client with a single API key — no Aeon required. This path is in development.

When it launches, the flow will be:

1. Connect a whitelisted wallet at **dashboard.houndflow.com** and generate your API key.
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

Until then, use **Path A**.

---

## Next steps

- [Skills reference](mcp-tools.md) — what each skill investigates
- [Configuration](configuration.md) — bring your own data source
- [Architecture](architecture.md) — how the skills and planned platform fit together
