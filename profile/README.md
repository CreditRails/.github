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

```
Wallet Activity (Stellar)
      │
      ▼
 Horizon Indexer    →  extracts payments, savings, remittances, repayments
      │
      ▼
 Scoring Engine     →  weighted model → score 300–850 + risk tier (A–D)
      │
      ▼
 Soroban Contract   →  score anchored on-chain, immutable audit trail
      │
      ▼
 DID Credential     →  W3C Verifiable Credential issued to the user
      │
      ▼
 Apps & Protocols   →  read score via API or direct contract call
```

Connect a wallet once. Any lending protocol, payments app, or anchor can query a credit profile with a single call — no forms, no bank statements, no intermediaries.

---

## Repositories

| Repo | Description |
|---|---|
| [**contracts**](https://github.com/CreditRails/contracts) | Soroban smart contracts — `credit_score` and `credential_registry` |
| [**indexer**](https://github.com/CreditRails/indexer) | Streams Horizon ledger data, runs the scoring model, writes scores on-chain |
| [**frontend**](https://github.com/CreditRails/frontend) | Wallet-connected dashboard + admin panel |
| [**sdk**](https://github.com/CreditRails/sdk) | Client SDK for querying scores and verifying credentials |

Reference docs (architecture, API, scoring model, Stellar/Soroban notes) live in [`data/`](https://github.com/CreditRails/.github/tree/main/data) in this repo.

---

## Score factors

| Factor | Weight | What it measures |
|---|---|---|
| Payment History | 30% | On-time, late, and missed payments |
| Transaction Volume | 22% | Consistent monthly financial activity |
| Account Age | 18% | How long the wallet has been active |
| DeFi Participation | 12% | Engagement with Blend, AMMs, and DeFi protocols |
| Credit Diversity | 10% | Range of transaction types |
| Wallet Health | 8% | Asset diversification and balance consistency |

## Blend integration

Score sets **interest rate** and **max borrow limit** on Blend pools — higher score, lower rate, larger limit. Every repayment updates the score on-chain, building a flywheel: *score → better rate → repayment → better score.*

| Score | Tier | Rate (USDC Pool) | Max Borrow |
|---|---|---|---|
| 800+ | A · Excellent | 3.0% | $20,000 |
| 740–799 | B+ · Very Good | 5.2% | $12,500 |
| 670–739 | B · Good | 7.8% | $6,000 |
| 580–669 | C · Fair | 11.5% | $2,500 |

---

## API

```
GET  /score/{wallet}        → credit score + risk tier
GET  /credential/{wallet}   → W3C Verifiable Credential (JWT)
POST /verify                → verify a user-submitted credential
```

<div align="center">
  <sub>Built on Stellar. Powered by on-chain behavior.</sub>
</div>
