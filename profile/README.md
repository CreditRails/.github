<div align="center">
  <img src="../assets/logo.png" alt="CreditRails" width="88" />

  # CreditRails

  **Portable on-chain credit infrastructure for the open economy.**

  CreditRails turns Stellar financial activity into a verifiable credit profile — one users carry across every app, without ever exposing raw transaction history.

  [![Twitter](https://img.shields.io/badge/twitter-@credit__rails-1DA1F2?logo=x&logoColor=white)](https://twitter.com/credit_rails)
  [![Stellar](https://img.shields.io/badge/built%20on-Stellar-000000?logo=stellar)](https://stellar.org)
</div>

---

## The problem

1.7 billion people transact daily — remittances, payroll, savings — with no credit history a bank can read. On Stellar, every payment is public and verifiable, but nothing turns that activity into a credit signal. The result: DeFi lending stays stuck at 150%+ overcollateralization, unable to tell a first-time wallet from one with three years of clean repayment history.

## The solution

CreditRails reads what's already public on Stellar and turns it into a credit signal no bank statement ever provided. An indexer streams a wallet's full Horizon history — payments, savings behavior, remittance patterns, DeFi repayments — and a transparent, weighted scoring model converts that activity into a 300–850 score with a risk tier, using the same seven factors for every wallet, mainnet or testnet.

The score doesn't just sit in a database. It's written on-chain through a Soroban contract, so it's auditable and tamper-evident rather than something CreditRails could quietly change. From there it's issued as a W3C Verifiable Credential the user actually holds — not a record CreditRails keeps custody of — so they can present it anywhere without re-exposing their raw transaction history each time.

Connect a wallet once. Any lending protocol, payments app, or anchor can read a credit profile from the API or the contract directly — no forms, no bank statements, no intermediary re-underwriting the same person from scratch.

---

## What's live today

- **Real-time scoring on testnet** — the indexer pulls a wallet's full Horizon history (payments, trades, operations, effects) and computes a live 300–850 score, no mocked data.
- **Blend DeFi position detection** — live reads against Blend lending pools surface active collateral, supply, and liability positions as part of the score.
- **On-chain score commit** — a deployed `credit_score` Soroban contract on testnet can be written to directly from the indexer, anchoring a wallet's score on-chain.
- **Wallet-connected dashboard** — connect a Stellar wallet and see your real score, factor breakdown, and transaction history computed live, not a demo.
- **Client SDK** (`@creditrails/sdk`) — TypeScript client for the real scoring API.

Building next: W3C Verifiable Credential issuance (contract exists, not yet wired to the indexer), mainnet deployment, a live price oracle, and Blend score→loan-terms mapping.

---

## Repositories

| Repo | Description |
|---|---|
| [**contracts**](https://github.com/CreditRails/contracts) | Soroban smart contracts — `credit_score` and `credential_registry` |
| [**indexer**](https://github.com/CreditRails/indexer) | Streams Horizon ledger data, runs the scoring model, writes scores on-chain |
| [**frontend**](https://github.com/CreditRails/frontend) | Wallet-connected dashboard + admin panel |
| [**sdk**](https://github.com/CreditRails/sdk) | Client SDK for querying scores |

Reference docs (architecture, API, scoring model, Stellar/Soroban notes) live in [`data/`](https://github.com/CreditRails/.github/tree/main/data) in this repo.

---

## Score factors

Seven weighted factors, each 0–100, mapped onto the 300–850 score:

| Factor | Weight | What it measures |
|---|---|---|
| Payment History | 27% | Regularity/consistency of inflows and outflows |
| Transaction Volume | 18% | Total USD moved in + out, log-scaled |
| Account Age | 13% | Days since the wallet's first observed activity |
| Savings Trend | 13% | Net accumulation vs. total flow |
| DeFi Participation | 11% | Active lending positions + protocol diversity (Blend, Soroswap) |
| Remittance Regularity | 9% | Recurring same-counterparty inflows (payroll/remittance pattern) |
| Diversity | 9% | Distinct assets + counterparties touched |

Full formula and per-factor math: [`data/scoring-methodology.md`](https://github.com/CreditRails/.github/blob/main/data/scoring-methodology.md).

## Blend integration

Score-aware lending is the target design: higher score → lower rate → larger limit → repayment → better score. Today, the indexer already detects a wallet's live Blend positions (collateral, supply, liabilities) as a scoring input — the score→loan-terms mapping on the Blend side is the next piece being built, not yet live.

---

## API

The real, running indexer API (see [`indexer`](https://github.com/CreditRails/indexer)):

```
GET  /api/public/score/{wallet}?network=   → public, unauthenticated dry-run score
GET  /api/score/{wallet}?network=          → same, admin-token gated
POST /api/score/{wallet}/commit?network=   → writes the score on-chain (testnet only)
```

No credential-issuance or verification endpoint exists yet — see "Building next" above.

<div align="center">
  <sub>Built on Stellar. Powered by on-chain behavior.</sub>
</div>
