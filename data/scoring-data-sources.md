# Scoring Data Sources

Reference for what APIs the indexer (`indexer/`) actually calls to build a wallet's signals, and what each response field feeds into the score. Keep this in sync with `indexer/src/horizon.ts`, `indexer/src/signals.ts`, and `indexer/src/config.ts` (the single source of truth for weights/thresholds).

---

## Network

| | URL |
|---|---|
| Horizon (testnet) | `https://horizon-testnet.stellar.org` |
| Soroban RPC (testnet) | `https://soroban-testnet.stellar.org` |
| Passphrase | `Test SDF Network ; September 2015` |

Defined in `indexer/src/config.ts` → `NETWORK`.

---

## Endpoints used

### `GET /accounts/{wallet}/payments`
`indexer/src/horizon.ts` → `fetchPayments()`

Paginated (`order=asc&limit=200`, follows `_links.next`). Includes `payment`, `path_payment_strict_send/receive`, `create_account`, `account_merge` operations.

Feeds:
- **Account age** — timestamp of the first record
- **Activity frequency** (`txPerWeek`) — count over the lookback window
- **Inflow / outflow USD volume** — `to === wallet` vs `from === wallet`, amount × static price (`ASSET_USD_PRICE` in config)
- **Large payment events** — any single payment ≥ `THRESHOLDS.largeAmountUsd`
- **Recurring counterparties** (payroll/remittance regularity) — inflow grouped by `from` address, checks repeat count + gap-day stddev
- **Asset/counterparty diversity** — distinct `asset_code` and distinct addresses touched

### `GET /accounts/{wallet}/trades`
`indexer/src/horizon.ts` → `fetchTrades()`

Paginated the same way. This is the authoritative "swap executed" signal (as opposed to `manage_buy_offer`/`manage_sell_offer` on `/operations`, which only records an *order placed*, not filled).

Feeds:
- **Large swap events** — `max(base_amount, counter_amount)` in USD ≥ `THRESHOLDS.largeAmountUsd`
- **Activity frequency** — counted alongside payments
- **Asset diversity** — base/counter asset codes

### `GET /accounts/{wallet}` (existence check only)
`indexer/src/horizon.ts` → `accountExists()`

Used to short-circuit unfunded/unknown wallets before pulling history.

---

## Not yet wired up

- **Soroban RPC `getEvents`** against the Blend pool contract, for on-chain loan repayment history (on-time/late/missed). Planned per `data/integrations.md` — highest-weight signal once added, currently no weight in `FACTOR_WEIGHTS`.
- **Live price oracle** — `ASSET_USD_PRICE` in `indexer/src/config.ts` is a static placeholder (XLM=$0.10, USDC=$1). Fine for testnet; mainnet needs a real feed.

---

## Where the actual judging parameters live

All weights, thresholds, and tier cutoffs are centralized in **`indexer/src/config.ts`** — nothing else in the indexer should hardcode a number that affects the score. See `FACTOR_WEIGHTS` and `THRESHOLDS` there.
