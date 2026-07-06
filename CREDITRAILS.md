# CreditRails — Build Plan

## What We Are Building

CreditRails is a portable on-chain credit infrastructure that reads Stellar financial activity and converts it into a verifiable credit profile users can carry across any application.

---

## Core Components

### 1. Stellar Transaction Indexer
- Connects to Stellar Horizon API to stream and index ledger activity
- Tracks payroll deposits, remittance inflows, savings patterns, payment frequency, and wallet behavior
- Stores structured financial history per wallet address
- Runs continuously — every new transaction updates the user profile

### 2. Behavioral Credit Scoring Engine
- Ingests indexed transaction data per wallet
- Applies a weighted scoring model across signals:
  - Payment consistency and frequency
  - Remittance regularity and volume
  - Savings growth over time
  - Payroll stability
  - Wallet age and activity depth
- Outputs a credit score, risk tier, and lending eligibility flag
- Model is updatable as new signal types are added

### 3. Verifiable Credit Credential (W3C DID)
- Issues a signed credential tied to the user's Stellar wallet
- Credential contains score, risk tier, and eligibility — not raw data
- User controls the credential and chooses when to share it
- Built on W3C Decentralized Identifier and Verifiable Credential standards
- Privacy-preserving — lenders see the result, not the underlying history

### 4. Credit API Layer
- REST API for lenders, fintechs, and protocols to query credit data
- Endpoints:
  - `GET /score/{wallet}` — returns credit score and risk tier
  - `GET /credential/{wallet}` — returns verifiable credential
  - `POST /verify` — verifies a credential submitted by a user
- Rate-limited and key-authenticated for enterprise access
- Subscription and per-query pricing tiers

### 5. Blend Protocol Integration
- Connects CreditRails credit scores to Blend's lending pools
- Allows Blend to offer undercollateralized loans based on CreditRails risk assessment
- Risk API feeds directly into Blend's borrower evaluation flow
- Enables lenders to set loan terms dynamically based on score tier

### 6. User Dashboard
- Wallet-connected interface for users to view their credit profile
- Shows score, contributing signals, and credential status
- Allows users to share credentials with lenders directly

---

## Technology Stack

- **Blockchain:** Stellar (Horizon API for indexing, Soroban for on-chain credential anchoring if needed)
- **Indexer:** Node.js or Python service consuming Stellar Horizon event stream
- **Scoring Engine:** Python — statistical model, updatable weights
- **Credential Layer:** W3C DID + Verifiable Credentials (JSON-LD)
- **API:** REST (Node.js / FastAPI)
- **Database:** PostgreSQL for structured financial history
- **Infrastructure:** Cloud-hosted, horizontally scalable indexer

---

## Integrations

| Integration | Purpose |
|---|---|
| Stellar Horizon | Transaction indexing |
| Blockroll | Payroll signal ingestion |
| Sava | Savings behavior signals |
| SeevCash / Fiatsend / VANK | Remittance signals |
| Blend | Undercollateralized lending |

---

## Build Phases

### Phase 1 — Scoring Foundation
- Stellar Horizon indexer live on testnet
- Behavioral scoring engine producing scores from wallet history
- Internal API returning score and risk tier per wallet

### Phase 2 — Credential Issuance
- W3C DID credential issuance live
- User-owned credential containing score and eligibility
- Credential verification endpoint for lenders

### Phase 3 — Blend Integration
- CreditRails risk API connected to Blend lending pools
- Lenders can query borrower eligibility before loan issuance
- Undercollateralized loan flow tested end-to-end on testnet

### Phase 4 — Mainnet Pilot
- Indexer and scoring engine live on mainnet
- Real users generating real credit profiles
- At least one active lending integration
- Open SDK published for ecosystem integrations

---

## Success Metrics

- Scoring engine live and producing scores on testnet
- Verifiable credentials issued and verifiable
- At least one lending protocol integration complete
- Measurable loan originations in mainnet pilot
- Open-source SDK available for new integrations
