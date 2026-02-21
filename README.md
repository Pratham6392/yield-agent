# YieldAgent by Yield.xyz

[![ClawHub](https://img.shields.io/badge/ClawHub-yield--agent-red)](https://clawhub.ai/apurvmishra/yield-agent)

**The yield layer for the agent era.**

2,988 yield opportunities. 75+ chains. One unified interface. Staking, lending, vaults, restaking, and liquidity pools — all via the Yield.xyz API. Secure, controlled access to on-chain yield for agents.

Non-custodial. Schema-driven. Agent-native.

---

### Core Capabilities

| Capability | Description |
|-----------|-------------|
| **Discover** | Query yields across every protocol and chain |
| **Enter** | Build unsigned transactions to deposit — your wallet signs |
| **Track** | View balances, accrued interest, pending actions |
| **Manage** | Claim rewards, restake, redelegate |
| **Exit** | Withdraw in one command |

---

## Quick Start

### Install

```bash
npx clawhub@latest install yield-agent
```

Or manually:
```bash
git clone https://github.com/stakekit/yield-agent.git ~/.openclaw/skills/yield-agent
chmod +x ~/.openclaw/skills/yield-agent/scripts/*.sh
```

### Use

```bash
# Find yields
./scripts/find-yields.sh base USDC

# Inspect a yield's schema
./scripts/get-yield-info.sh base-usdc-aave-v3-lending

# Enter a position
./scripts/enter-position.sh base-usdc-aave-v3-lending 0xYOUR_ADDRESS '{"amount":"100"}'

# Check balances
./scripts/check-portfolio.sh base-usdc-aave-v3-lending 0xYOUR_ADDRESS
```

A free shared API key is included in `skill.json`. For production, get your own from [dashboard.yield.xyz](https://dashboard.yield.xyz) or set `YIELDS_API_KEY` env var.

---

## Scripts

| Script | Endpoint | Description |
|--------|----------|-------------|
| `find-yields.sh` | `GET /v1/yields` | Discover yields by network and token |
| `get-yield-info.sh` | `GET /v1/yields/{id}` | Inspect yield schema, limits, tokens |
| `list-validators.sh` | `GET /v1/yields/{id}/validators` | List validators for staking |
| `enter-position.sh` | `POST /v1/actions/enter` | Enter a yield position |
| `exit-position.sh` | `POST /v1/actions/exit` | Exit a yield position |
| `manage-position.sh` | `POST /v1/actions/manage` | Claim, restake, redelegate |
| `check-portfolio.sh` | `POST /v1/yields/{id}/balances` | Check balances and pending actions |

---

## Project Structure

```
yield-agent/
├── SKILL.md                          # Main skill definition (agent reads this)
├── skill.json                        # Manifest, API config, triggers
├── scripts/                          # 7 bash scripts wrapping the API
├── references/
│   ├── openapi.yaml                  # OpenAPI spec (source of truth for types)
│   ├── safety.md                     # Safety checks and guardrails
│   ├── superskill.md                 # 40 advanced agent capabilities
│   ├── chain-formats.md              # Unsigned tx formats per chain
│   ├── wallet-integration.md         # Wallet setup and signing flow
│   └── examples.md                   # Agent conversation patterns
```

---

## Architecture: Layer by Layer

The repository is organized into four layers, from outermost configuration to innermost reference material.

---

### Layer 1 — Manifest (`skill.json`)

The machine-readable configuration file. Agents and runtimes (OpenClaw, ClawHub) read this first.

| Field | Purpose |
|-------|---------|
| `name`, `version`, `description` | Package identity |
| `api.baseUrl`, `api.authHeader`, `api.apiKey` | How to reach the Yield.xyz API |
| `openclaw.triggers` | Keywords that activate this skill ("stake", "lend", "APY", …) |
| `openclaw.requires.bins` | Runtime dependencies (`curl`, `jq`) |
| `sbom.files` | Which files the agent must read (SKILL.md, openapi.yaml, safety.md) |
| `state` | Where persistent state is written (`state/yield-cache.json`, `state/yield-safety.json`) |

---

### Layer 2 — Skill Definition (`SKILL.md`)

The natural-language instruction manual the agent reads at runtime. It contains:

- **YAML frontmatter** — tool definitions with names, descriptions, entry-point scripts, and argument schemas. This mirrors `skill.json` but in the human-readable SKILL format.
- **Critical rules** — never modify `unsignedTransaction`, always fetch the yield schema before acting, always submit the tx hash after broadcasting, execute transactions in `stepIndex` order.
- **Quick-start patterns** — copy-paste command sequences for the four main flows: Enter → Track → Manage → Exit.
- **API endpoint table** — a quick reference for all 11 API endpoints.
- **Pointers to references** — tells the agent where to find the OpenAPI spec, chain formats, wallet guide, examples, and superskills.

---

### Layer 3 — Scripts (`scripts/`)

Seven Bash scripts. Each is a thin, validated wrapper around one Yield.xyz API call. They share a common structure:

1. **Locate `skill.json`** — searches four candidate directories so the scripts work whether installed globally or run locally.
2. **Load credentials** — `YIELDS_API_KEY` env var overrides the key in `skill.json`; same for `YIELDS_API_URL`.
3. **Validate inputs** — sanitize path parameters against an allowlist of safe characters; validate JSON with `jq`; validate numeric arguments with regex.
4. **Call the API** — one `curl` invocation per script.
5. **Format output** — pipe through `jq`; action scripts append a per-transaction instruction block reminding the agent to sign → broadcast → submit-hash → poll.

| Script | HTTP Method + Endpoint | Purpose |
|--------|------------------------|---------|
| `find-yields.sh` | `GET /v1/yields` | Discover yields by network and/or token, with pagination |
| `get-yield-info.sh` | `GET /v1/yields/{id}` | Inspect a single yield: schema, limits, token, entry/exit args |
| `list-validators.sh` | `GET /v1/yields/{id}/validators` | List validators for staking yields |
| `enter-position.sh` | `POST /v1/actions/enter` | Build unsigned deposit transactions |
| `exit-position.sh` | `POST /v1/actions/exit` | Build unsigned withdrawal transactions |
| `manage-position.sh` | `POST /v1/actions/manage` | Build unsigned claim/restake/redelegate transactions |
| `check-portfolio.sh` | `POST /v1/yields/{id}/balances` | Fetch balances and pending actions for a position |

---

### Layer 4 — References (`references/`)

Read-on-demand documentation the agent consults for specifics. Each file has a narrow scope:

| File | What it covers |
|------|----------------|
| `openapi.yaml` | Source of truth for every DTO, enum, request shape, and response shape in the API |
| `safety.md` | Pre-execution checklist (schema check → balance check → status check → network check → amount format → tx order), configurable guardrails (spend limits, protocol allowlist, TVL/APY thresholds), and the golden rule: build transactions, never modify them |
| `chain-formats.md` | How `unsignedTransaction` is encoded for each chain family (EVM JSON, Cosmos hex, Solana hex/base64, Substrate JSON object, Tezos hex, TON JSON, Near hex, Sui base64, Aptos base64 BCS, Cardano CBOR, Stellar XDR, Tron JSON) and which SDK signs each |
| `wallet-integration.md` | Compatible wallet skills (Crossmint, Privy, Portal, Turnkey), the sign → broadcast → submit-hash → poll loop, and setup instructions |
| `examples.md` | 10 annotated agent conversations (discovery, deposit, balance check, claim, withdrawal, cross-chain comparison, rebalance, validator staking, swap-then-yield, safety guardrails), plus a smoke-test checklist |
| `superskill.md` | 40 advanced capabilities built on top of the core scripts: rate alerts, trend detection, morning briefings, cross-chain comparison, portfolio diversification, rotation workflows, reward harvesting, and scheduled checks — all requiring only agent memory and scheduling |

---

### How the Layers Interact

```
User / Agent runtime
        │
        ▼
  skill.json  ──────────────────────────────────  SKILL.md
  (machine config: triggers,                      (agent instructions:
   API key, tool definitions)                      rules, patterns, endpoint table)
        │                                               │
        └──────────────┬────────────────────────────────┘
                       ▼
              scripts/*.sh   (validate → call API → format output)
                       │
                       ▼
            Yield.xyz REST API
                       │
                       ▼
       references/  (consulted on demand for types, chain formats,
                     safety rules, wallet setup, and examples)
```

---

## Key Rules

1. **Always fetch the yield schema before calling an action** — the API is self-documenting
2. **Amounts are human-readable** — `"100"` = 100 USDC, `"1"` = 1 ETH
3. **Always submit the tx hash after broadcasting** — `PUT /v1/transactions/{txId}/submit-hash`
4. **Never modify `unsignedTransaction`** — sign exactly what the API returns
5. **Execute transactions in `stepIndex` order** — wait for CONFIRMED between each

---

## Requirements

- `curl` and `jq`
- A wallet for signing (Crossmint, Portal, Turnkey, Privy, or any compatible wallet)

---

## Security

Yield.xyz is **SOC 2 compliant** ([trust.yield.xyz](https://trust.yield.xyz/)). A safe, controlled environment for AI agents to access on-chain yields.

---

## Links

- [agent.yield.xyz](https://agent.yield.xyz)
- [ClawHub](https://clawhub.ai/apurvmishra/yield-agent)
- [GitHub](https://github.com/stakekit/yield-agent)
- [API Docs](https://docs.yield.xyz)
- [API Recipes](https://github.com/stakekit/api-recipes)
- [Get API Key](https://dashboard.yield.xyz)
- [Yield.xyz](https://yield.xyz)

---

## Important Notice & Risk Disclosure

YieldAgent is a software tool designed to help users discover yield opportunities and construct transactions using the Yield.xyz infrastructure. It is not a financial advisor, broker, dealer, or fiduciary. Yield.xyz does not provide financial, investment, tax, accounting, or legal advice. Nothing in this repository, within the YieldAgent interface, or in any related materials constitutes a recommendation, solicitation, endorsement, or offer to buy, sell, hold, or otherwise transact in any digital asset or to pursue any particular investment strategy.

All actions taken through YieldAgent are initiated and executed at your sole discretion. You are fully responsible for evaluating and understanding the risks involved, including but not limited to smart contract vulnerabilities, protocol failures, counterparty exposure, market volatility, liquidity constraints, loss of private keys, technical errors, and changing regulatory requirements. Digital assets and decentralized finance involve substantial risk, including the potential for total loss of funds. Only use funds you can afford to lose. You should conduct your own research and consult qualified professional advisors before making financial decisions.

By using YieldAgent, you acknowledge and accept these risks.

## License

Apache 2.0
