# Horizon Deprecation — Alternatives & Self-Hosting

Horizon is slated for deprecation. This doc tracks where CreditRails currently
depends on it, what replacing it would take, and the candidate alternatives.

## Where we use Horizon today

- `indexer/src/horizon.ts` — dedicated REST client hitting:
  - `/accounts/{wallet}` (existence check)
  - `/accounts/{wallet}/payments`
  - `/accounts/{wallet}/trades`
  - `/accounts/{wallet}/operations` (also used for "first op ever" → account age)
  - `/accounts/{wallet}/effects`
- Consumed by `compute.ts`, `signals.ts`, `defi.ts` — i.e. the entire scoring
  pipeline runs on Horizon data, not Soroban RPC.
- `indexer/src/config.ts` — `horizonUrl` for testnet/mainnet, plus
  `archivalHorizonUrl` (Lobstr mirror) because public mainnet Horizon
  truncates history to 1 year (SDF policy since Aug 2024).
- `frontend/src/lib/wallet.ts` — standalone `HORIZON_URL` (testnet), appears
  otherwise unused in frontend.

## Self-hosting Horizon

**Components:**
- `stellar-core` in **captive-core** mode (Horizon manages it in-process, no
  separate core DB needed).
- Horizon's own ingestion process + Postgres — this is what builds the
  `/payments`, `/trades`, `/operations`, `/effects` endpoints; it's not a
  proxy to core, it computes these from raw ledger changes.
- Access to **history archives** (SDF publishes to S3; community mirrors also
  exist) — captive-core pulls from these during catchup.

**The expensive part — full-history catchup:**
- Full mainnet history since genesis → multi-TB DB, catchup can run for days
  from cold start.
- SDF and community providers publish periodic DB snapshots to skip replaying
  from genesis — use one instead of a cold catchup.

**Ongoing ops:**
- Core/Horizon versions must track Stellar protocol upgrades — falling
  behind risks ingestion breaking on a new protocol version.
- Continuous disk growth, backups, ingestion-lag monitoring.

**The catch:** self-hosting fixes availability/rate-limits/1yr-truncation,
but does **not** fix Horizon-the-software going unmaintained against future
protocol versions — that risk is inherent to running a sunsetting product,
self-hosted or not.

## Alternatives (least → most rebuild work)

| Option | What it is | Fit |
|---|---|---|
| Third-party hosted Horizon-compatible endpoints | More full-history mirrors alongside Lobstr (via Ankr, QuickNode, etc.) | Zero code change, only fixes single-point-of-failure on SDF's public instance |
| **Mercury** (xycloo) | Indexing/subscription service purpose-built for Stellar/Soroban account & contract history | Closest semantic match to our per-account payments/trades/effects queries — best first spike |
| Soroban RPC directly | `getTransactions`, events, `getLedgerEntries` | No aggregated payments/trades/effects endpoints — would require parsing ops ourselves |
| **Galexie** + custom indexer | SDF's raw ledger-meta exporter (dumps to GCS/S3) | Most future-proof, most work — effectively rebuild just the parts of Horizon we need, no retention limits |
| **Hubble** (BigQuery ledger data) | SDF's analytics pipeline | Good for batch scoring, too high-latency for live per-wallet queries |

**Recommendation:** spike Mercury first (least rebuild, matches our
`horizon.ts` shape); fall back to Galexie + custom ingestion if it doesn't
cover something we need. Track provider list for RPC in
[stellar-rpc-providers.md](stellar-rpc-providers.md).
