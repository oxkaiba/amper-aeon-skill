<p align="center">
  <img src="https://app.amper.chat/logos/amper-black-white.png" alt="Amper" height="60" />
  <span>&nbsp;&nbsp;&nbsp;&nbsp;</span>
  <img src="https://github.com/aaronjmars/aeon/blob/main/assets/aeon.jpg?raw=true" alt="Aeon" height="60" />
</p>

<h1 align="center">Amper x Aeon</h1>

<p align="center"><strong>DeFi execution layer for Aeon agents.</strong> One API, 80+ tools, autonomous on-chain operations across multiple chains.</p>

---

## What this is

An [Aeon](https://github.com/aaronjmars/aeon) skill that connects to the [Amper](https://app.amper.chat) API, giving your agent access to portfolio reads, token swaps, lending, perpetual futures, options trading, bridging, and more. No custom code required. Drop the skill in, add your API key, schedule it.

## What your agent can do

| Category | Protocol | Tools |
|----------|----------|-------|
| **Portfolio** | Wallet, ERC20 | Balances, token details, analytics |
| **Swaps** | Coinbase CDP, Aori | Token swaps, cross-chain swaps |
| **Lending** | Aave v3 | Supply, withdraw, borrow, repay, position checks |
| **Vaults** | Morpho | Deposit, withdraw, best vaults by APY, claim rewards |
| **Perps** | Avantis | Open/close positions (up to 500x), limit orders, TP/SL |
| **Options** | Derive | Options trading, subaccount management |
| **Bridges** | Relay, Aori | Cross-chain token transfers across supported chains |
| **Staking** | Venice | Stake, mint, burn, claim rewards |
| **Data** | DeFiLlama, Pyth | Protocol TVL, yields, price feeds |

Full tool list with parameter schemas available at `GET /api/v1/tools`.

## Quick start

### 1. Get an Amper account

Sign up at [app.amper.chat](https://app.amper.chat) with your email or wallet. An embedded wallet is created for you automatically.

### 2. Subscribe

An active Amper subscription is required for API access. Two options:

| Method | Cost |
|--------|------|
| **USDC** | $6/month on Base |
| **NFT** | Mint the Amper subscription NFT |

Subscribe from **Settings > Subscription** in the app.

### 3. Generate an API key

Go to **Settings > API** and create a key:

| Key type | Access |
|----------|--------|
| **Read** | Balances, positions, quotes, analytics |
| **Trade** | Everything in Read + swaps, lending, perps, options, bridging |

The key is shown once. Copy it and store it securely.

### 4. Add the skill to Aeon

Copy the `SKILL.md` file into your Aeon `skills/amper-defi/` directory:

```
skills/
  amper-defi/
    SKILL.md
```

### 5. Add your secret

Add `AMPER_API_KEY` to your GitHub Actions secrets (Settings > Secrets > Actions):

```
AMPER_API_KEY=amp_live_abc123...
```

### 6. Register in aeon.yml

```yaml
amper-defi:
  enabled: true
  schedule: "0 */6 * * *"    # every 6 hours, or whatever you want
  var: ""                     # empty = portfolio overview
```

## Usage examples

### Portfolio overview (default)

```yaml
amper-defi:
  enabled: true
  schedule: "0 8 * * *"
  var: ""
```

When `var` is empty, the skill runs a full portfolio health check: wallet balances, Aave positions, Morpho vaults, unclaimed rewards.

### Swap tokens

```yaml
amper-defi:
  enabled: true
  schedule: workflow_dispatch
  var: "swap 100 USDC to ETH on base"
```

### Check yields

```yaml
amper-defi:
  enabled: true
  schedule: "0 */12 * * *"
  var: "compare my current Morpho vault APY against the best available vaults"
```

### Open a leveraged position

```yaml
amper-defi:
  enabled: true
  schedule: workflow_dispatch
  var: "open a 10x long ETH/USD position with 50 USDC collateral on Avantis"
```

### Chain with other skills

```yaml
chains:
  defi-morning:
    schedule: "0 7 * * *"
    steps:
      - parallel: [token-movers, amper-defi]
      - skill: morning-brief
        consume: [token-movers, amper-defi]
```

## API details

| | |
|---|---|
| **Base URL** | `https://app.amper.chat/api/v1` |
| **Auth** | Bearer token (`amp_live_` prefix) |
| **Execute** | `POST /execute` with `{ "tool": "...", "params": { ... } }` |
| **Discover** | `GET /tools` returns all tools with JSON schemas |
| **Rate limits** | 30 req/min, 500 req/day per key |
| **Execution** | Sequential (one request at a time per key) |
| **Idempotency** | Required `X-Idempotency-Key` header (UUID, unique per request) |
| **Timeouts** | 60s for trade tools, 30s for read tools |

## Security

- **No transfers.** The API cannot send tokens out of your wallet. Funds only move within DeFi protocols (swaps, lending, vaults, perps).
- **Key hashing.** Only the SHA-256 hash of your key is stored. Plaintext is never persisted.
- **Scope control.** Read keys cannot execute trades. Trade keys cannot do transfers.
- **Rate limiting.** Sliding window per key prevents runaway execution.
- **Sequential lock.** One operation at a time per key. No parallel execution races.

## Error handling

The skill receives structured error responses from the API:

| Error | HTTP | Meaning |
|-------|------|---------|
| `rate_limit` | 429 | Too many requests. `retry_after` field tells you when to retry. |
| `busy` | 429 | Another request is still executing. Wait and retry. |
| `scope_denied` | 403 | Key type doesn't have access to this tool. |
| `subscription_required` | 403 | Amper subscription expired. |
| `execution_failed` | 500 | Tool execution failed. Check the `message` field. |

Full error reference in the [Amper API docs](https://app.amper.chat).

## Links

- [Amper App](https://app.amper.chat)
- [Aeon Framework](https://github.com/aaronjmars/aeon)
- [Amper on X](https://x.com/helloamper)
