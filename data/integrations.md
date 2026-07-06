# Integrations

CreditRails ingests signals from Stellar-native apps. Each integration adds a new signal type to the scoring model.

---

## Active Integrations


### Stellar (Horizon API)
- **Signal:** Full ledger history — payments, swaps, deposits, withdrawals
- **How:** REST polling + SSE stream via `https://horizon.stellar.org`
- **Weight:** Foundation — all wallets get base signals from this

### Blend
- **Site:** https://blend.capital
- **Signal:** Loan repayment behavior — on-time, late, missed
- **How:** Blend emits on-chain events; indexer watches pool contract operations
- **Weight:** High — loan repayment is the strongest creditworthiness signal

---

## Planned Integrations

### Bitso
- **Site:** https://bitso.com
- **Signal:** LATAM exchange payroll deposits and savings activity
- **Integration type:** Horizon payment stream filtered by Bitso anchor accounts

### Sava
- **Signal:** Savings account growth and consistency
- **Integration type:** On-chain savings contract events

### Fiatsend
- **Signal:** Recurring remittance inflows
- **Integration type:** Horizon payment stream filtered by Fiatsend anchor accounts

### MoneyGram
- **Site:** https://moneygram.com
- **Signal:** Cross-border remittance inflows via MoneyGram's Stellar integration
- **Integration type:** Horizon payment stream filtered by MoneyGram anchor accounts

---

## Adding a New Integration

Any Stellar-native app can become a signal source via the CreditRails SDK:

```typescript
import { CreditRailsSDK } from '@creditrails/sdk';

const cr = new CreditRailsSDK({ apiKey: 'sk_live_...' });

// Submit a signal event
await cr.signals.submit({
  wallet: 'GBZX...4K9P',
  type: 'payroll_deposit',
  amount_usdc: 1200,
  timestamp: new Date().toISOString(),
  source: 'my-app-id',
});
```

Signal types: `payroll_deposit`, `savings_deposit`, `remittance_inflow`, `loan_repayment`, `bill_payment`
