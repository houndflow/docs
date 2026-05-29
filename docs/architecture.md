# Architecture

Hound Flow is delivered in two layers: a **skill pack** that runs today inside the [Aeon](https://github.com/aaronjmars/aeon) framework, and a **hosted platform** (MCP server + dashboard) in development. The skill pack is the current delivery; the platform is additive.

## Current delivery: the Hound skill pack

Each skill is a self-contained `SKILL.md` — markdown instructions an agent executes. Skills read onchain data from Base via public endpoints or your own BYOK keys, and return a structured verdict. They carry no infrastructure of their own: they run wherever the Aeon agent runs.

```
┌─────────────────────────────────────────────┐
│  Aeon agent (your fork / clone)              │
│  ┌─────────────────────────────────────────┐ │
│  │  Hound skills (SKILL.md)                 │ │
│  │  rug-scan · contract-audit · …           │ │
│  └───────────────────┬─────────────────────┘ │
└──────────────────────┼───────────────────────┘
                       │  BYOK or public endpoints
                       ▼
        Base data sources: Basescan · Etherscan v2 · RPC
```

- **Install:** `./add-skill houndflow/hound-skills <skill>` (see [Getting Started](getting-started.md)).
- **Data sources:** resolved per call from the agent's environment — Basescan key → Etherscan v2 key → custom RPC → public fallback (see [Configuration](configuration.md)).
- **Read-only:** skills never sign or send transactions.

## Planned platform

For users who want investigation as a managed service rather than running an Aeon agent.

```
┌──────────────────────────────────────────────────────┐
│  AI Agent (Cursor · Claude Code · Codex · any MCP)    │
└───────────────────────────┬──────────────────────────┘
                            │  MCP (Streamable HTTP) + Bearer key   [PLANNED]
                            ▼
┌──────────────────────────────────────────────────────┐
│  mcp.houndflow.com — hosted MCP server     [PLANNED]  │
│  • Validates API key                                  │
│  • Exposes the same investigation tools               │
│  • BYOK passed per call, never stored                 │
└───────────────────────────┬──────────────────────────┘
                            ▼
        Base data sources: Basescan · Etherscan v2 · RPC

dashboard.houndflow.com — profile management   [IN DEVELOPMENT]
  • Wallet (SIWE) login, whitelist gate
  • Manage one API key per account
  • Terminal UI for agent / AI sessions   [PLANNED]
```

### Hosted MCP server (planned)

A remote MCP server over Streamable HTTP, authenticated per request with a Hound API key. It exposes the same investigation tools as the skill pack. BYOK credentials, when supplied, are used in-memory for a single call and never stored. There is no anonymous access.

### Dashboard (in development)

A profile-management surface — not a data dashboard:

- Connect a wallet (SIWE — Sign-In With Ethereum); access is whitelisted during early access.
- Generate, rotate, and revoke a single API key per account.
- A built-in **terminal UI** for agent / AI sessions is planned.

The backend stores only accounts (wallet), API-key hashes, and the whitelist. It does not store data-source secrets — BYOK is supplied by the client per call.

## Design principles

- **Standalone skills.** The skill pack works in any Aeon agent today; the platform is optional and additive.
- **Client-side BYOK.** Users own their data-source secrets; the platform never stores them.
- **Read-only.** Tools never sign or send transactions. No `onchain_writes`.
- **Spec-driven.** Each skill's `SKILL.md` is the behavior spec, so the skill pack and the planned hosted tools stay in sync.

## Roadmap

| Phase | Focus |
|-------|-------|
| Now | Hound skill pack for Aeon — 6 core skills on Base, BYOK or public endpoints |
| Next | Hosted MCP server (`mcp.houndflow.com`); profile dashboard; more skills (fund-flow, linked-wallets, approvals, honeypot, full reports) |
| Later | Dashboard terminal UI for agent sessions; multi-chain (Ethereum, Arbitrum, Optimism); SDKs |
