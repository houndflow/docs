# Architecture

Hound Flow is an onchain-investigation platform for **Base**. AI agents connect to the Hound MCP server — hosted or run locally — and invoke read-only investigation tools with a single Hound API key. Users bring their own data-source credentials (BYOK); Hound Flow never stores them.

## System overview

```
┌─────────────────────────────────────────────────────────┐
│  AI Agent  (Cursor · Claude Code · Codex · any MCP)      │
└──────────────┬──────────────────────────┬───────────────┘
   HOSTED mode │                           │ LOCAL mode
   MCP (HTTP)  │                           │ MCP (stdio, npx)
   Bearer key  │                           │ keys from env
   + BYOK hdrs ▼                           ▼ (never leave machine)
┌──────────────────────────┐   ┌──────────────────────────┐
│  mcp.houndflow.com        │   │  @houndflow/mcp (local)   │
│  • Validates Hound key    │   │  • Runs on user machine   │
│  • Exposes hound_* tools  │   │  • Same hound_* tools     │
│  • BYOK in-memory only,   │   │  • Calls Base sources     │
│    never stored           │   │    directly               │
└──────┬─────────────┬──────┘   └─────────────┬────────────┘
       │ validate    │ analyze (+BYOK,        │ analyze (+BYOK,
       │ key         │ per-call)              │ in-process)
       ▼             ▼                        ▼
┌──────────────┐  ┌──────────────────────────────────────┐
│ api.houndflow│  │  Base data sources (user's BYOK or     │
│ .com         │  │  keyless fallback)                     │
│ accounts,    │  │  Basescan · Etherscan v2 · custom RPC  │
│ keys, w-list │  └──────────────────────────────────────┘
└──────────────┘

dashboard.houndflow.com — wallet login + profile (API key). No data-source storage.
```

## Components

### 1. MCP Server (`hound-mcp`)

The product surface, shipped as one codebase with two transports:

- **Hosted** (`mcp.houndflow.com`) — MCP over **Streamable HTTP**. Authenticates every request via `Authorization: Bearer <api_key>` and rejects anonymous calls. BYOK credentials arrive as request headers, are held **in memory for the single call only**, and are never written anywhere.
- **Local** (`npx @houndflow/mcp`) — MCP over **stdio**, running on the user's machine. BYOK credentials come from the user's environment and **never leave their machine** (zero-transit).

Both register the same `hound_*` tools, validate inputs (EVM address / tx-hash shape), and return structured results with a `source=` footer.

### 2. Backend API (`api.houndflow.com`)

The account layer (not a data-source store).

- **Auth & accounts** — SIWE (Sign-In With Ethereum) login, whitelist gate, one API key per account (stored as a hash).
- **Key validation** — `internal/keys/validate` lets the hosted MCP server check a key (Redis-cached).
- **Analyzers** — Investigation logic for the six tools. In hosted mode the MCP server may proxy to these endpoints, passing the per-request BYOK config (which is used and discarded, never stored). In local mode the same logic runs in-process.
- **Rate-limit & metering** — Per-account limits and usage counts.

**What the backend stores:** accounts (wallet), API-key hashes, whitelist. **It does not store data-source secrets** — there is no data-source table.

### 3. Dashboard (`dashboard.houndflow.com`)

Minimal by design. Connect wallet (SIWE), pass the whitelist gate, and manage your **profile**: view your wallet, generate / rotate / revoke your single API key, and copy a ready-to-paste MCP client snippet. **There is no data-source configuration page** — BYOK is set in your own client config.

## Authentication model

```
Wallet → SIWE signature → session (dashboard only)
                            │
                            ▼
                   Whitelist check ── not listed ──▶ request access
                            │ listed
                            ▼
                   Generate API key (one per account)
                            │
                            ▼
        Agent uses API key → MCP server → tools
```

- **SIWE** proves wallet ownership with an off-chain signature (no gas).
- **Whitelist** gates platform access during MVP.
- **API key** authenticates agent/tool traffic. Exactly one active key per account; rotating revokes the previous key.

## Data flow: a tool call (hosted mode)

1. Agent calls `hound_detect_rug_risk(token)` over MCP with its Bearer key and any BYOK headers.
2. MCP server validates the key (Redis-cached) and resolves the account.
3. The data layer fetches onchain data using the **per-request BYOK credentials** (Basescan → Etherscan v2 → custom RPC → keyless fallback). The credentials live in memory for this call only.
4. The rug analyzer scores the token and returns a structured verdict (with `source=`).
5. The MCP server returns the verdict; usage is metered. BYOK credentials are discarded — never persisted or logged.

In **local mode**, steps 3–4 run entirely on the user's machine and the BYOK credentials never touch Hound Flow infrastructure.

## Design principles

- **Client-side BYOK.** Users own their data-source secrets. Hound Flow never stores them — hosted mode holds them in memory for one call; local mode keeps them on the user's machine.
- **Read-only.** Tools never sign or send transactions. No `onchain_writes`.
- **Authenticated by default.** The hosted MCP endpoint has no anonymous access.
- **Minimal custody.** The backend stores only accounts, API-key hashes, and the whitelist.
- **Spec-driven tools.** Each tool's behavior is documented and versioned, so hosted and local implementations stay in sync.

## Roadmap

| Phase | Focus |
|-------|-------|
| MVP | Base-only, 6 core tools, wallet whitelist, client-side BYOK, hosted + local MCP |
| Next | Fund-flow tracing, linked-wallet clustering, approval & honeypot checks, full investigation reports |
| Later | Multi-chain (Ethereum, Arbitrum, Optimism), SDKs, deeper ecosystem integrations |
