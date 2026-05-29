# MCP Tools Reference

Hound Flow exposes six investigation tools over MCP. All operate on **Base** (chain id `8453`), are **read-only**, and return structured, agent-readable results. Connect first — see [Getting Started](getting-started.md).

## Conventions

- **Inputs** are EVM addresses (`0x` + 40 hex chars) or transaction hashes (`0x` + 64 hex chars).
- **Outputs** include a human-readable verdict plus structured fields, and a footer noting which data source answered (`source=basescan|etherscan|rpc`).
- Tools never sign or send transactions.

---

## `hound_detect_rug_risk`

Assess rug-pull risk for a token.

| | |
|--|--|
| **Input** | `token` — token contract address |
| **Returns** | Risk verdict (`LOW` / `ELEVATED` / `HIGH` / `CRITICAL`) with a weighted score and itemized red flags / mitigants |

Signals include: source verification, mint authority, blacklist/freeze, pausable transfers, mutable fees, ownership renouncement, proxy/upgradeability, LP lock status, and top-holder concentration.

*Example:* "Use hound to scan rug risk for `0x…` on Base."

---

## `hound_analyze_contract`

Audit a smart contract's powers and structure.

| | |
|--|--|
| **Input** | `address` — contract address |
| **Returns** | Verification status, proxy/upgradeability pattern, ownership/admin roles, and a capability matrix of dangerous functions (mint, blacklist, pause, upgrade, fund-drain) with whether each is live and owner-gated |

Detects proxy patterns (EIP-1967 Transparent, UUPS, Beacon) and flags backdoors such as arbitrary `call`/`delegatecall` or token-rescue functions.

*Example:* "Audit contract `0x…` on Base."

---

## `hound_analyze_wallet`

Build a behavioral profile of a wallet.

| | |
|--|--|
| **Input** | `address` — wallet address |
| **Returns** | Age, activity class (`bot` / `whale` / `sniper` / `trader` / `holder` / `deployer`), funding source, top counterparties, and risk flags |

Resolves the funding origin (CEX / bridge / DEX / unknown EOA) and flags interactions with previously flagged contracts and live approvals to unverified spenders.

*Example:* "Profile wallet `0x…`."

---

## `hound_check_deployer_history`

Map every contract a deployer has shipped.

| | |
|--|--|
| **Input** | `address` — deployer address (or a token, whose creator is resolved first) |
| **Returns** | List of deployed contracts with each one's fate (`ALIVE` / `ABANDONED` / `RUGGED`), pattern linkage (reused templates), and a serial-rugger verdict |

*Example:* "What else did `0x…` deploy on Base?"

---

## `hound_decode_transaction`

Turn a raw transaction into a plain-English story.

| | |
|--|--|
| **Input** | `tx_hash` — transaction hash |
| **Returns** | Decoded method, token movements with USD values, counterparty labels, net effect per address, and suspicious-approval flags |

Recognizes common actions (transfer, approve, swaps) and surfaces unlimited approvals to unverified spenders.

*Example:* "Explain transaction `0x…`."

---

## `hound_analyze_token_holders`

Analyze a token's holder distribution.

| | |
|--|--|
| **Input** | `token` — token contract address |
| **Returns** | Top-N share, concentration index (HHI), holders-to-50%, and a verdict (`HEALTHY` / `CONCENTRATED` / `FRAGILE`) |

Classifies and excludes non-circulating holders (LP, lockers, burn, contracts) and flags whale clusters that share a funding source.

*Example:* "Show holder concentration for `0x…`."

---

## Errors

| Error | Meaning |
|-------|---------|
| `401 unauthorized` | Missing, invalid, or revoked API key |
| `403 not_whitelisted` | Wallet not approved for access |
| `INVALID_INPUT` | Malformed address or tx hash |
| `DATA_SOURCE_ERROR` | All configured data sources failed — check [Configuration](configuration.md) |
| `RATE_LIMITED` | Account rate limit hit — add a BYOK key/RPC for higher limits |

## Roadmap tools

Planned for future releases: fund-flow tracing, linked-wallet clustering, approval audits, honeypot checks, LP-lock checks, MEV detection, and full composite investigation reports. See [Architecture → Roadmap](architecture.md#roadmap).
