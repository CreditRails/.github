# CreditRails

**Portable on-chain credit infrastructure for the open economy.**

CreditRails reads Stellar financial activity and converts it into a verifiable credit profile — one that users carry across every application, without exposing raw transaction history.

---

## What It Does

| Layer | What it does |
|---|---|
| **Stellar Indexer** | Streams ledger activity via Horizon API; extracts payment, savings, remittance, and payroll signals per wallet |
| **Scoring Engine** | Applies a weighted behavioral model to produce a credit score (300–850) and risk tier |
| **DID Credential** | Issues a signed W3C Verifiable Credential — users share the result, never raw data |
| **Credit API** | REST endpoints for lenders and protocols to query scores and verify credentials |
| **Blend Integration** | Feeds scores directly into Blend's undercollateralized lending pools |
| **User Dashboard** | Wallet-connected interface to view score, factors, transaction history, and credentials |

---

## The Problem

**1.7 billion people are unbanked.** Most of them transact daily — remittances, savings, payroll, peer-to-peer payments — but have no credit history that financial institutions can read. Traditional credit scores require bank accounts, credit cards, and formal loan histories that billions of people will never have.

On-chain finance on Stellar changes this. Every payment, repayment, savings deposit, and DeFi interaction is public, timestamped, and verifiable. But no protocol aggregates this into a credit layer that applications can actually consume.

The result: DeFi lending on Stellar requires 150%+ overcollateralization — locking out exactly the users who need credit most. Lenders can't differentiate between a first-time user and someone with three years of on-chain repayment history.

**CreditRails is the missing credit infrastructure layer.**

---

## The Solution

CreditRails reads on-chain behavioral signals, runs them through a weighted scoring model, anchors the result as a Soroban smart contract, and issues a W3C Verifiable Credential to the user.

```
Wallet Activity (Stellar)
       ↓
  Horizon Indexer  →  extracts payments, savings, remittances, repayments
       ↓
  Scoring Engine   →  weighted model → score 300–850 + risk tier (A–D)
       ↓
  Soroban Contract →  score anchored on-chain, immutable audit trail
       ↓
  DID Credential   →  W3C Verifiable Credential issued to the user
       ↓
  Apps & Protocols →  read score via API or direct contract call
```

Users connect a Stellar wallet once. Any app — lending protocol, payments platform, anchor — can query their credit profile with a single API call, without ever seeing raw transaction data.

**No forms. No bank statements. No intermediaries.**

---

## User Base

CreditRails targets three overlapping user groups:

**Unbanked and underbanked individuals (primary)**
People in emerging markets who receive remittances, earn payroll on-chain, or save in stablecoins but have no formal credit file. For these users, CreditRails is the first time their financial behavior has ever been recognized as creditworthy. Target regions: Southeast Asia, Sub-Saharan Africa, Latin America, South Asia.

**Crypto-native DeFi users**
Active Stellar wallet holders who participate in Blend, AQUA, StellarX, and other protocols. They have rich on-chain histories but no way to convert that reputation into lower borrowing costs. CreditRails unlocks undercollateralized access for users who have earned it.

**Protocols and lenders (B2B)**
Any Stellar-native lending protocol, anchor, or payments app that needs to assess counterparty risk without running KYC pipelines. CreditRails provides a programmable, verifiable credit signal via REST API or Soroban contract call.

---

## Market Size

| Segment | Size |
|---------|------|
| Global unbanked population | 1.7B people |
| Stellar active wallets (2026) | ~8M accounts |
| Global DeFi lending TVL | $50B+ |
| Stellar ecosystem TVL | $500M+ |
| Cross-border remittance volume | $860B / year |
| Underserved credit market (emerging markets) | $4.9T gap |

Even capturing a fraction of the Stellar-native credit demand — users who want to borrow against their on-chain reputation rather than lock collateral — represents a multi-million dollar addressable market in protocol fees, credential issuance, and API access tiers.

---

## Blend Integration

CreditRails feeds on-chain credit scores directly into Blend's lending pools, enabling undercollateralized borrowing based on behavioral history.

**How it works:**
- When you borrow on Blend, the pool reads your CreditRails score from the Soroban `credit_score` contract
- Your score sets your **interest rate** and **maximum borrow limit** — higher score = lower rate + larger limit
- Each repayment updates your score on-chain, building credit history over time

**Rates by tier:**

| Score | Tier | Rate (USDC Pool) | Max Borrow |
|-------|------|-----------------|------------|
| 800+ | A / Excellent | 3.0% | $20,000 |
| 740–799 | B+ / Very Good | 5.2% | $12,500 |
| 670–739 | B / Good | 7.8% | $6,000 |
| 580–669 | C / Fair | 11.5% | $2,500 |

This creates a flywheel: **score → better rates → repayment → better score**.

---

## Score Factors

| Factor | Weight | What it measures |
|--------|--------|-----------------|
| Payment History | 30% | On-time payments, late payments, missed payments |
| Transaction Volume | 22% | Consistent monthly volume signals financial activity |
| Account Age | 18% | How long the wallet has been active on Stellar |
| DeFi Participation | 12% | Engagement with Blend, AMMs, and DeFi protocols |
| Credit Diversity | 10% | Range of transaction types: payments, swaps, loans, savings |
| Wallet Health | 8% | Asset diversification and non-zero balance consistency |

---

## Industry Use Cases

**Lending protocols (Blend)**
Undercollateralized loans based on behavioral credit score. Borrowers with a repayment track record get better rates automatically — no collateral lockup required.

**Payroll + remittance apps (Blockroll, Fiatsend, VANK)**
Verified income signals from payroll deposits and recurring remittances feed into the scoring model — boosting scores for users who receive salary on-chain.

**Savings protocols (Sava)**
Consistent savings behavior is a scoring factor. Users who lock funds into Sava accumulate score improvements over time.

**DeFi AMMs and DEXes (StellarX, AQUA)**
Protocols can gate fee tiers or liquidity incentives behind a minimum CreditRails score, rewarding long-term participants.

**Anchor platforms (MoneyGram, SDF)**
KYC-light onboarding by requiring a minimum CreditRails credential instead of full document verification, for lower-risk operations.

**Cross-border payments**
Users building a payment history across currencies and corridors carry a portable reputation into any new service — even ones that have never seen their home bank.

---

## API

```
GET  /score/{wallet}       → credit score + risk tier
GET  /credential/{wallet}  → W3C Verifiable Credential (JWT)
POST /verify               → verify a user-submitted credential
```

One call. No bank statements. No forms.

---

Built on Stellar. Powered by on-chain behavior.
