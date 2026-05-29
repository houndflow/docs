# Skills Reference

Hound Flow ships six onchain investigation skills. All operate on **Base** (chain id `8453`), are **read-only**, and return structured, agent-readable results.

Install them into an Aeon agent (see [Getting Started](getting-started.md)). When the hosted platform launches, each skill will also be exposed as an MCP tool under the name in the **Planned MCP tool** column.

## Conventions

- **Inputs** are EVM addresses (`0x` + 40 hex chars) or transaction hashes (`0x` + 64 hex chars), passed as the skill's variable input.
- **Outputs** include a human-readable verdict plus structured fields, and a footer noting which data source answered (`source=basescan|etherscan|rpc`).
- Skills never sign or send transactions.

| Skill | Planned MCP tool |
|-------|------------------|
| `rug-scan` | `hound_detect_rug_risk` |
| `contract-audit` | `hound_analyze_contract` |
| `wallet-profile` | `hound_analyze_wallet` |
| `deployer-trace` | `hound_check_deployer_history` |
| `tx-explain` | `hound_decode_transaction` |
| `holder-concentration` | `hound_analyze_token_holders` |

---

## `rug-scan`

Assess rug-pull risk for a token.

| | |
|--|--|
| **Input** | token contract address |
| **Returns** | Risk verdict (`LOW` / `ELEVATED` / `HIGH` / `CRITICAL`) with a weighted score and itemized red flags / mitigants |

Signals include: source verification, mint authority, blacklist/freeze, pausable transfers, mutable fees, ownership renouncement, proxy/upgradeability, LP lock status, and top-holder concentration.

---

## `contract-audit`

Audit a smart contract's powers and structure.

| | |
|--|--|
| **Input** | contract address |
| **Returns** | Verification status, proxy/upgradeability pattern, ownership/admin roles, and a capability matrix of dangerous functions (mint, blacklist, pause, upgrade, fund-drain) with whether each is live and owner-gated |

Detects proxy patterns (EIP-1967 Transparent, UUPS, Beacon) and flags backdoors such as arbitrary `call`/`delegatecall` or token-rescue functions.

---

## `wallet-profile`

Build a behavioral profile of a wallet.

| | |
|--|--|
| **Input** | wallet address |
| **Returns** | Age, activity class (`bot` / `whale` / `sniper` / `trader` / `holder` / `deployer`), funding source, top counterparties, and risk flags |

Resolves the funding origin (CEX / bridge / DEX / unknown EOA) and flags interactions with previously flagged contracts and live approvals to unverified spenders.

---

## `deployer-trace`

Map every contract a deployer has shipped.

| | |
|--|--|
| **Input** | deployer address (or a token, whose creator is resolved first) |
| **Returns** | List of deployed contracts with each one's fate (`ALIVE` / `ABANDONED` / `RUGGED`), pattern linkage (reused templates), and a serial-rugger verdict |

---

## `tx-explain`

Turn a raw transaction into a plain-English story.

| | |
|--|--|
| **Input** | transaction hash |
| **Returns** | Decoded method, token movements with USD values, counterparty labels, net effect per address, and suspicious-approval flags |

Recognizes common actions (transfer, approve, swaps) and surfaces unlimited approvals to unverified spenders.

---

## `holder-concentration`

Analyze a token's holder distribution.

| | |
|--|--|
| **Input** | token contract address |
| **Returns** | Top-N share, concentration index (HHI), holders-to-50%, and a verdict (`HEALTHY` / `CONCENTRATED` / `FRAGILE`) |

Classifies and excludes non-circulating holders (LP, lockers, burn, contracts) and flags whale clusters that share a funding source.

---

## Common end-states

Skills follow Aeon conventions — they log a status and notify only when a finding warrants it:

| State | Meaning |
|-------|---------|
| `*_OK` | Ran, nothing noteworthy — logged, no notification |
| `*_FLAGGED` | A risk/verdict crossed the alert threshold — notifies |
| `*_NO_TARGET` | No address/hash provided — exits cleanly |
| `*_ERROR` | All data sources failed — see [Configuration](configuration.md) |

## Roadmap skills

Planned: fund-flow tracing, linked-wallet clustering, approval audits, honeypot checks, LP-lock checks, MEV detection, and full composite investigation reports. See [Architecture → Roadmap](architecture.md#roadmap).
