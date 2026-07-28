# Scoring Data Sources

Reference for what APIs the indexer (`indexer/`) actually calls to build a wallet's signals, and what each response field feeds into the score. Keep this in sync with `indexer/src/horizon.ts`, `indexer/src/defi.ts`, `indexer/src/signals.ts`, and `indexer/src/config.ts` (the single source of truth for weights/thresholds).

---

## Network

Every network-dependent call goes through `NETWORKS[network]` in `indexer/src/config.ts` — `testnet` or `mainnet`, selected via `--network` (CLI) or `?network=` (API).

| | Testnet | Mainnet |
|---|---|---|
| Horizon | `https://horizon-testnet.stellar.org` | `https://horizon.stellar.org` |
| Horizon (full history) | — | `https://horizon.stellar.lobstr.co` (`archivalHorizonUrl`) |
| Soroban RPC | `https://soroban-testnet.stellar.org` | `https://rpc.ankr.com/stellar_soroban` |
| Passphrase | `Test SDF Network ; September 2015` | `Public Global Stellar Network ; September 2015` |

**Public mainnet Horizon truncates history to 1 year** (SDF policy since Aug 2024). Wallets older than that will get `accountAgeConfidence: "estimated"` unless queried against the archival LOBSTR endpoint instead.

No `credit_score` contract is deployed on mainnet yet — mainnet scoring is dry-run only (`CONTRACT.mainnet` in `config.ts` is `null`).

---

## Endpoints used

### `GET /accounts/{wallet}/payments`
`indexer/src/horizon.ts` → `fetchPayments()`

Paginated (`order=asc&limit=200`, follows `_links.next`). Includes `payment`, `path_payment_strict_send/receive`, `create_account`, `account_merge` operations.

Feeds: activity frequency, inflow/outflow USD volume, large payment events, recurring counterparties, asset/counterparty diversity.

### `GET /accounts/{wallet}/trades`
`indexer/src/horizon.ts` → `fetchTrades()`

The authoritative "swap executed" signal (as opposed to `manage_buy_offer`/`manage_sell_offer` on `/operations`, which only records an *order placed*, not filled).

Feeds: large swap events, activity frequency, asset diversity.

### `GET /accounts/{wallet}/operations`
`indexer/src/horizon.ts` → `fetchOperations()`

Every operation type — a superset of `/payments`: `invoke_host_function`, `liquidity_pool_deposit/withdraw`, `manage_buy_offer/manage_sell_offer`, `change_trust`, `claim_claimable_balance`, `create_account`, etc. Fetched **most-recent-first** (`order=desc`) and capped at `MAX_PAGES` (100 pages / 20k records) — high-activity wallets (market makers, bots) can have far more history than that, and it's the *recent* window (`LOOKBACK_DAYS`) that scoring actually needs, not the oldest records.

Feeds: **DeFi interaction detection** — `invoke_host_function` operations are decoded (`indexer/src/defi.ts` → `decodeInvocation()`) and matched against the DeFi contract registry.

### `GET /accounts/{wallet}/operations?order=asc&limit=1`
`indexer/src/horizon.ts` → `fetchFirstOperation()`

A single, cheap, uncapped request for the account's very first operation — decoupled from `fetchOperations()` specifically so exact wallet age doesn't depend on whether the capped, most-recent-first operations list happens to reach back that far.

Feeds: **Account age** — `accountAgeConfidence: "exact"` when this record is a `create_account` operation; `"estimated"` otherwise (either a mainnet wallet older than Horizon's 1-year retention window, or an account with no operation history at all).

### `GET /accounts/{wallet}/effects`
`indexer/src/horizon.ts` → `fetchEffects()`

Feeds: `contract_credited`/`contract_debited` effects add to inflow/outflow USD volume and large-event detection for Soroban token transfers that never appear as classic payments (no counterparty is exposed at the effect level, so these don't feed recurrence detection).

### `GET /accounts/{wallet}` (existence check only)
`indexer/src/horizon.ts` → `accountExists()`

Used to short-circuit unfunded/unknown wallets before pulling history.

### Soroban RPC `simulateTransaction` — DeFi detection
`indexer/src/defi.ts`

- `poolFactory.is_pool(contract_id)` — free read-only check against Blend's pool factory (`DEFI_CONTRACTS[network]`, role `poolFactoryV2`), used to classify `invoke_host_function` calls to contracts not already in the static registry (i.e. Blend pools created after this file was last updated). Confirmed pools are cached to `indexer/data/pool-registry.<network>.json`.
- `pool.get_positions(wallet)` — free read-only call against every known Blend pool (`indexer/src/defi.ts` → `fetchBlendPositions()`), returning live collateral/liability/supply state. Unaffected by Horizon/RPC history-retention limits, unlike operation- or event-based detection — this is the primary "does this wallet have skin in the game on Blend" signal.

### DeFi contract registry
`indexer/src/config.ts` → `DEFI_CONTRACTS`

Static, verified addresses for Blend (pool factory, backstop, emitter, Comet BLND:USDC pool) and Soroswap (router, factory), per network. Sourced from `docs.blend.capital/mainnet-deployments`, `github.com/blend-capital/blend-utils`, and `github.com/soroswap/core/public/*.contracts.json`.

---

## Not yet wired up

- **Live price oracle** — `ASSET_USD_PRICE` in `indexer/src/config.ts` is a static placeholder (XLM=$0.10, USDC=$1) on both networks. Fine for testnet; mainnet needs a real feed (e.g. the Reflector Soroban oracle) before mainnet USD-denominated signals (volume, large events, Blend position sizing) are trustworthy. Not addressed yet — `blendPositions` currently reports collateral/liability/supply as booleans rather than USD amounts for this reason.
- **Repayment timeline (on-time/late/missed)** — `get_positions()` gives current state, not history. Scoring the *behavior* of repayment (not just "has a position") would need to index Blend pool `submit()` events over time.

---

## Where the actual judging parameters live

All weights, thresholds, and tier cutoffs are centralized in **`indexer/src/config.ts`** — nothing else in the indexer should hardcode a number that affects the score. See `FACTOR_WEIGHTS` and `THRESHOLDS` there.
