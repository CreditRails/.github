# Blend Integration

## Overview

Blend is a decentralized lending protocol on Stellar. CreditRails feeds on-chain credit scores directly into Blend's lending pools, enabling undercollateralized borrowing based on behavioral credit history rather than locked collateral.

---

## How It Works

```
User's Stellar wallet activity
         ↓
  CreditRails scoring engine
         ↓
  credit_score Soroban contract (on-chain)
         ↓
  Blend pool reads score on borrow request
         ↓
  Rate + limit set based on score tier
         ↓
  Repayment recorded → CreditRails updates score
```

**Score → Blend eligibility/rate**, and **repayment behavior → score update**.

---

## Rate Tiers

| Score | Tier | Borrow Rate (USDC Pool) | Max Borrow | Max LTV |
|-------|------|------------------------|------------|---------|
| 800–850 | A / Excellent | 3.0% | $20,000 | 85% |
| 740–799 | B+ / Very Good | 5.2% | $12,500 | 75% |
| 670–739 | B / Good | 7.8% | $6,000 | 65% |
| 580–669 | C / Fair | 11.5% | $2,500 | 55% |
| Below 580 | D / Poor | Not eligible | — | — |

---

## Blend Pools Available

### USDC Lending Pool
- Min score: 680
- Pool APY: 6.8%
- Rate for score 742: **5.2%**
- Max available: $8,500

### XLM Borrow Pool
- Min score: 700
- Pool APY: 4.2%
- Rate for score 742: **3.1%**
- Max available: 12,000 XLM

### USDT Stable Pool
- Min score: 720
- Pool APY: 7.1%
- Rate for score 742: **5.8%**
- Max available: $4,000

---

## Technical Integration

### Contract call flow

When a user initiates a borrow on Blend:

```rust
// Blend pool contract calls CreditRails credit_score contract
let profile = credit_score_client.get_score(&wallet_address);
let rate = calculate_rate(profile.score, profile.tier);
let max_borrow = calculate_max(profile.score, pool_liquidity);
```

### Score update on repayment

When a loan is repaid on Blend:

```rust
// Blend repayment triggers CreditRails score update
credit_score_client.update_score(&wallet, new_score);
// ScoreUpdated event emitted on-chain
```

### Contracts involved

| Contract | Address (Testnet) | Role |
|----------|-------------------|------|
| `credit_score` | TBD | Stores user score + profile |
| `credential_registry` | TBD | Anchors W3C credential |
| Blend Pool | TBD | Reads score, sets rate |

---

## The Flywheel

```
Higher score
     ↑
Better repayment history  ←──────────────┐
     ↑                                   │
Lower borrow rate                        │
     ↑                                   │
User borrows more (affordable)           │
     ↑                                   │
Uses Blend (more DeFi participation) ────┘
```

Consistent on-chain behavior creates a self-reinforcing credit cycle — exactly like traditional credit building, but fully on-chain and user-owned.

---

## Blend SDK Usage (TypeScript)

```typescript
import { CreditRails } from "@creditrails/sdk";
import { BlendPool } from "@blend-capital/sdk";

const cr = new CreditRails({ apiKey: process.env.CR_API_KEY });

// 1. Get user score
const { score, tier } = await cr.score.get(walletAddress);

// 2. Query eligible Blend pools
const pools = await BlendPool.eligible(walletAddress, score);

// 3. Simulate borrow
const simulation = await pools[0].simulateBorrow({
  amount: 5000,
  asset: "USDC",
  creditScore: score,
});

console.log(simulation.rate);     // e.g. "5.2%"
console.log(simulation.maxBorrow); // e.g. 8500
```

---

## Industry Context

Traditional undercollateralized lending requires trust-based relationships (KYC + bank statements). Blend + CreditRails replaces this with verifiable on-chain behavioral data:

- **No forms** — wallet activity IS the application
- **No bank statements** — Stellar transaction history IS the income proof
- **No intermediaries** — score is computed and anchored on-chain
- **Portable** — same credential works across every Blend pool and future protocols

This is the first undercollateralized lending system on Stellar that uses a behavioral credit layer instead of social capital or overcollateralization.
