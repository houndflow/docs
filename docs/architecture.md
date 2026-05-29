# Architecture

Hound Flow is delivered in two layers: **skills** that run inside the [Aeon](https://github.com/aaronjmars/aeon) framework (merged upstream in [#269](https://github.com/aaronjmars/aeon/pull/269)), and a **hosted platform** — a live OSINT chat app, MCP server, and dashboard. Use whichever fits; they share the same investigation logic.

## The skills (in Aeon)

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

## The platform

For users who want investigation as a managed service rather than running an Aeon agent. The **chat app, hosted MCP server, and dashboard are all live**.

```
┌───────────────────────────┐     ┌──────────────────────────────────────┐
│  chat.houndflow.com [LIVE] │     │  AI Agent (Cursor · Claude Code · …) │
│  Hound Agent OSINT chat    │     └──────────────────┬───────────────────┘
│  wallet login · auto tools │                        │ MCP (Streamable HTTP) + Bearer key
└─────────────┬──────────────┘                        ▼
              │                       ┌──────────────────────────────────────┐
              └──────────────────────▶│  mcp.houndflow.com — MCP server [LIVE]│
                                      │  • Validates API key                  │
                                      │  • Exposes the investigation tools     │
                                      │  • BYOK passed per call, never stored  │
                                      └──────────────────┬─────────────────────┘
                                                         ▼
                              Base data sources: Basescan · Etherscan v2 · RPC

houndflow.com/dashboard — profile management   [LIVE]
  • Wallet (SIWE) login, whitelist gate
  • Generate / rotate / revoke one API key per account
```

### Chat app (live)

`chat.houndflow.com` — **Hound Agent**, a wallet-gated OSINT chat. Sign in with a whitelisted wallet (SIWE), and the Hound MCP tools are auto-connected. Investigate in natural language; fund-flow diagrams render inline. Users can bring their own Basescan key / RPC via the BYOK page.

### Hosted MCP server (live)

A remote MCP server over Streamable HTTP, authenticated per request with a Hound API key. It exposes the investigation tools to any MCP client. BYOK credentials, when supplied, are used in-memory for a single call and never stored. There is no anonymous access.

### Dashboard (live)

A profile-management surface — not a data dashboard:

- Connect a wallet via **SIWE** (Sign-In With Ethereum) — an off-chain signature, no gas. Access is whitelisted during early access (non-whitelisted wallets are invited to the waitlist).
- Generate, rotate, and revoke a single API key per account. The key is shown in plaintext once at creation; the server stores only its hash.

The backend stores only accounts (wallet), API-key hashes, and the whitelist. It does not store data-source secrets — BYOK is supplied by the client per call.

## Design principles

- **Standalone skills.** The skills run in any Aeon agent (merged upstream in [#269](https://github.com/aaronjmars/aeon/pull/269)); the hosted platform is additive.
- **Client-side BYOK.** Users own their data-source secrets; the platform never stores them.
- **Read-only.** Tools never sign or send transactions. No `onchain_writes`.
- **Spec-driven.** Each skill's `SKILL.md` is the behavior spec, so the Aeon skills and the hosted tools stay in sync.

## Roadmap

| Phase | Focus |
|-------|-------|
| Now | Skills merged into Aeon; **live** chat app, hosted MCP server, and dashboard on Base |
| Next | More skills (linked-wallets, approvals, honeypot, full investigation reports); fund-flow upstreamed to Aeon |
| Later | Multi-chain (Ethereum, Arbitrum, Optimism); SDKs |
