# Architecture

Hound Flow is a hosted onchain-investigation platform for **Base**. AI agents connect to a remote MCP server with a single API key; the server validates the key and delegates analysis to a backend that reads onchain data through the account's configured data sources.

## System overview

```
┌─────────────────────────────────────────────────────────┐
│  AI Agent  (Cursor · Claude Code · Codex · any MCP)      │
└───────────────────────────┬─────────────────────────────┘
                            │  MCP (Streamable HTTP)
                            │  Authorization: Bearer <api_key>
                            ▼
┌─────────────────────────────────────────────────────────┐
│  mcp.houndflow.com — MCP Server                          │
│  • Validates API key (cached)                            │
│  • Exposes hound_* tools                                 │
│  • Thin proxy → backend                                  │
└───────────────────────────┬─────────────────────────────┘
                            │  internal HTTPS
                            ▼
┌─────────────────────────────────────────────────────────┐
│  api.houndflow.com — Backend                             │
│  • Auth: SIWE login, whitelist, API-key store            │
│  • Analyzer services (rug, contract, wallet, …)          │
│  • Per-account BYOK data-source resolver                 │
│  • Cache + rate-limit + usage metering                   │
└──────┬───────────────────────────────────┬──────────────┘
       │                                   │
       ▼                                   ▼
┌──────────────┐                  ┌──────────────────────┐
│ Postgres     │                  │ Base data sources     │
│ Redis        │                  │ Basescan / Etherscan  │
│              │                  │ v2 / custom RPC       │
└──────────────┘                  └──────────────────────┘

dashboard.houndflow.com — wallet login, API key, data-source config
```

## Components

### 1. MCP Server (`mcp.houndflow.com`)

The product surface. A stateless server speaking MCP over **Streamable HTTP** (not stdio — clients connect to a hosted URL, not a local binary).

- Authenticates every request via `Authorization: Bearer <api_key>`; rejects anonymous calls.
- Registers the `hound_*` tools and validates tool inputs (EVM address / tx-hash shape).
- Resolves the calling account from the key and forwards to the backend with that account's context.
- Maps backend results to MCP responses; surfaces errors without leaking internals.

### 2. Backend API (`api.houndflow.com`)

The intelligence and account layer.

- **Auth & accounts** — SIWE (Sign-In With Ethereum) login, whitelist gate, one API key per account (stored as a hash).
- **Analyzers** — One service per tool, implementing the documented heuristics for rug risk, contract audit, wallet profile, deployer trace, transaction decode, and holder concentration.
- **Data-source resolver** — Selects the account's configured source in priority order (see [Configuration](configuration.md)).
- **Cache, rate-limit, metering** — Redis-backed caching for read-only lookups; per-account rate limits and usage counts.

### 3. Dashboard (`dashboard.houndflow.com`)

The human entry point: connect wallet (SIWE), pass the whitelist gate, generate/rotate the API key, configure data sources, and copy a ready-to-paste MCP client snippet.

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
        Agent uses API key → MCP server → backend
```

- **SIWE** proves wallet ownership with an off-chain signature (no gas).
- **Whitelist** gates platform access during MVP.
- **API key** authenticates agent/tool traffic. Exactly one active key per account; rotating revokes the previous key.

## Data flow: a tool call

1. Agent calls `hound_detect_rug_risk(token)` over MCP with its Bearer key.
2. MCP server validates the key (Redis-cached) and resolves the account.
3. Server forwards the request to `api.houndflow.com/v1/rug/:token` with account context.
4. Backend checks cache; on miss, the resolver fetches onchain data via the account's BYOK source (Basescan → Etherscan v2 → custom RPC → keyless fallback).
5. The rug analyzer scores the token and returns a structured verdict.
6. MCP server returns the verdict to the agent; usage is metered.

## Design principles

- **Read-only.** Analyzers never sign or send transactions. No `onchain_writes`.
- **Authenticated by default.** The MCP endpoint has no anonymous access.
- **BYOK first.** Accounts use their own data sources; the shared keyless path is a rate-limited fallback.
- **Secrets protected.** API keys are hashed; BYOK credentials are encrypted at rest and never returned by any endpoint.
- **Spec-driven tools.** Each tool's behavior is documented and versioned, so the hosted backend and any open skill implementation stay in sync.

## Roadmap

| Phase | Focus |
|-------|-------|
| MVP | Base-only, 6 core tools, wallet whitelist, BYOK data sources |
| Next | Fund-flow tracing, linked-wallet clustering, approval & honeypot checks, full investigation reports |
| Later | Multi-chain (Ethereum, Arbitrum, Optimism), SDKs, deeper ecosystem integrations |
