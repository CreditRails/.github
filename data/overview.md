# CreditRails — Overview

CreditRails is portable, on-chain credit infrastructure built on Stellar.

It reads real financial activity from the Stellar blockchain — payments, remittances, savings, payroll deposits — and converts that behavioral history into a verifiable credit score. Users own a W3C Decentralized Identifier (DID) credential containing their score. They carry it across any application without exposing raw transaction data.

---

## The Problem

Billions of people are creditworthy but credit-invisible. They pay bills, send remittances, maintain savings, and receive payroll — but these signals are siloed in fintech apps, never aggregated, and never portable. Every lender starts from zero.

## The Solution

CreditRails indexes what's already happening on Stellar:
- **Blockroll** payroll deposits → income stability signal
- **Sava** savings activity → financial discipline signal
- **Fiatsend / VANK** remittances → recurring cash flow signal
- **Blend** repayments → credit behavior signal

These signals feed a scoring engine. The output is a score + a signed credential. Lenders query the credential — they never see the raw history.

---

## Key Properties

| Property | How |
|---|---|
| **Portable** | Score lives in a user-owned W3C VC, not in a database |
| **Privacy-preserving** | Lenders see the score, not the transactions |
| **Real-time** | Every confirmed Stellar ledger event can update the score |
| **Verifiable** | Any party can verify the credential without contacting CreditRails |
| **Open** | SDK available for any Stellar-native app to become a signal source |
