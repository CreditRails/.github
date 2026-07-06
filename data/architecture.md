# Architecture

## System Components

```
Stellar Horizon API
        │
        ▼
┌─────────────────────┐
│  Transaction Indexer │   streams ledger events per wallet
│  (Node.js / Python)  │   extracts: payments, savings, payroll, remittances
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   Scoring Engine     │   weighted behavioral model → score (300–850)
│   (Python)           │   outputs: score, risk tier, loan eligibility
└──────────┬──────────┘
           │
     ┌─────┴──────┐
     ▼            ▼
┌─────────┐  ┌──────────────────────┐
│  REST   │  │  Credential Issuer    │
│  API    │  │  (W3C DID / JWT VC)  │
└────┬────┘  └──────────┬───────────┘
     │                  │
     ▼                  ▼
Lenders / Protocols    User Wallet (owns credential)
(Blend, fintechs)      (shares with whoever they choose)
```

## Data Flow

1. User connects Stellar wallet
2. Indexer reads full ledger history via Horizon API
3. Scoring engine produces score + risk tier
4. Credential issuer wraps result in signed W3C VC
5. User's dashboard shows score, factors, and credential
6. User optionally shares credential with a lender
7. Lender calls `/verify` to confirm authenticity — no raw data exposed

## Database Schema (simplified)

```
wallet_profiles
  - wallet_address (PK)
  - score           INT
  - risk_tier       ENUM('A','B','C','D')
  - loan_eligible   BOOL
  - last_updated    TIMESTAMP

transaction_signals
  - wallet_address  FK
  - signal_type     ENUM('payroll','remittance','savings','repayment','payment')
  - amount          DECIMAL
  - timestamp       TIMESTAMP
  - source_app      VARCHAR  -- e.g. 'blockroll', 'sava', 'fiatsend'

credentials
  - wallet_address  FK
  - credential_id   VARCHAR
  - jwt             TEXT
  - issued_at       TIMESTAMP
  - expires_at      TIMESTAMP
  - score_snapshot  INT
```
