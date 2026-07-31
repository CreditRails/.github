# Credit Scoring Methodology

How a wallet's on-chain history turns into a score. This is the "what factors,
what weight, what formula" doc — for what Horizon/RPC calls the numbers come
from, see [scoring-data-sources.md](scoring-data-sources.md). Single source of
truth for every number below is `indexer/src/config.ts` — nothing else in the
indexer should hardcode a weight or threshold.

Verified live against real testnet data on 2026-07-31: an active wallet
(329 real txns) scored 683/tier C; a freshly funded wallet with 1 txn hit the
cold-start floor (300/tier F). Both round-tripped through the real Horizon →
`computeForWallet` → `scoreWallet` pipeline, no mocked values.

---

## 1. Score range & tiers

- Range: **300–850** (`SCORE_MIN`/`SCORE_MAX`), matches `contracts/credit_score/src/lib.rs` on-chain enforcement.
- Tiers (`TIER_CUTOFFS`, must match `score_to_tier` in the contract):

| Score | Tier |
|---|---|
| 800–850 | A |
| 740–799 | B |
| 670–739 | C |
| 580–669 | D |
| 300–579 | F |

- `percentile = round(((score - 300) / (850 - 300)) * 100)` — a linear position in the range, not a population percentile (no cross-wallet comparison happens yet).

## 2. Cold start rule

If `txCount < 3` (`THRESHOLDS.minTxForConfidentScore`), we don't have enough signal to score confidently: the wallet is clamped to `coldStartScore = SCORE_MIN` (300, tier F) and `coldStart: true` is returned. Factors are still computed and returned for transparency, but they don't drive the score in this case.

**Why 3:** below that, "regular payment history" and "recurring counterparty" detection are statistically meaningless — a single payment can't be distinguished from a fluke.

## 3. Final score formula

```
avg   = Σ (factor_value × factor_weight)         // 0-100, weights sum to 1
score = round(300 + (avg / 100) × (850 - 300))
tier  = score_to_tier(score)
```

Each `factor_value` is itself 0–100, clamped. `weightedFactorAverage()` in `indexer/src/scorer.ts` does the sum; `computeFactors()` in `indexer/src/factors.ts` computes each factor.

## 4. Factors, weights, and how each is computed

| Factor | Weight | What it measures |
|---|---|---|
| `paymentHistory` | **27%** | Regularity/consistency of inflows and outflows |
| `transactionVolume` | **18%** | Total USD moved (in + out), log-scaled |
| `accountAge` | **13%** | Days since first observed activity |
| `savingsTrend` | **13%** | Net accumulation vs. total flow — is the wallet building a balance or just passing money through? |
| `defiParticipation` | **11%** | Active lending positions + protocol diversity (Blend, Soroswap, ...) |
| `remittanceRegularity` | **9%** | Recurring same-counterparty inflows (payroll/remittance pattern) |
| `diversity` | **9%** | Distinct assets + counterparties touched |

Weights sum to 1.00.

### `paymentHistory` (0–100)
```
60  if hasRegularRecurrence
+ min(40, recurringCounterpartyCount × 10)
+ min(20, (txPerWeek / 3) × 20)                  // 3 = highActivityTxPerWeek
```

### `transactionVolume` (0–100)
Log-scaled so early volume matters more than marginal volume at the high end:
```
logScale(inflowUsd + outflowUsd, 10_000)         // 10,000 = volumeUsdForMaxScore
= log10(1 + value) / log10(1 + 10_000) × 100
```

### `accountAge` (0–100)
```
clamp(accountAgeDays / 730 × 100)                // 730 days (2yr) = ageDaysForMaxScore
```
`accountAgeDays` confidence is `"exact"` when the wallet's actual `create_account` operation was found, `"estimated"` otherwise (mainnet wallet older than Horizon's 1yr retention window, or no history at all).

### `savingsTrend` (0–100)
```
net       = inflowUsd - outflowUsd
totalFlow = inflowUsd + outflowUsd
savingsTrend = 0                                 if totalFlow == 0
             = clamp(((net / totalFlow) × 0.5 + 0.5) × 100)   otherwise
```
A wallet that only spends what it receives scores ~50; one accumulating a growing balance scores higher.

### `remittanceRegularity` (0–100)
```
70  if hasRegularRecurrence
+ min(30, recurringCounterpartyCount × 10)
```
`hasRegularRecurrence` requires ≥3 repeat inflows from the same counterparty (`minRecurrencesForRegularity`) with gaps regular enough (stddev ≤ 5 days, `maxGapStdDevDaysForRegularity`) to look like payroll/remittance rather than coincidence.

### `diversity` (0–100)
```
clamp(((distinctAssets + distinctCounterparties) / 10) × 100)   // 10 = diversityCountForMaxScore
```

### `defiParticipation` (0–100)
```
60  if any Blend position has collateral, supply, or liabilities
+ min(20, defiInteractionCount × 5)
+ (defiProtocolsTouched.length / 3) × 20         // 3 = defiProtocolsForMaxScore
```
An **active** lending position (collateral/supply/liabilities on Blend) counts far more than a bare "called the contract once" — repayment behavior is the strongest signal here, once we can see it (see §5).

---

## 5. Known gaps (not yet wired into scoring)

- **No live price oracle.** `ASSET_USD_PRICE` is a static placeholder (XLM=$0.10, USDC=$1) on both networks. Fine for testnet demo purposes; every USD-denominated factor (`transactionVolume`, `savingsTrend`, large-event detection, Blend position sizing) is unreliable on mainnet until this is a real feed (e.g. Reflector).
- **No repayment-timeline signal.** `defiParticipation` currently only sees *current* Blend position state (has collateral / has liabilities), not a history of on-time vs. late `submit()` calls. The single strongest lending-credit signal (do they pay back on time) isn't in the score yet.
- **No cross-wallet percentile.** `percentile` today is a linear position in the fixed 300–850 range, not a population-relative ranking.

## 6. Where this lives in code

- `indexer/src/config.ts` — every weight, threshold, tier cutoff, score range (change numbers here only).
- `indexer/src/factors.ts` — `computeFactors()`, the seven formulas above.
- `indexer/src/scorer.ts` — `scoreWallet()`, `tierForScore()`, cold-start clamp, final weighted-average formula.
- `contracts/credit_score/src/lib.rs` — on-chain mirror of the score range + tier cutoffs; must stay in sync with `config.ts` by hand (no shared source between Rust and TS yet).
