# API Reference

Base URL: `https://api.creditrails.xyz/v1`  
Auth: `Authorization: Bearer <api_key>`

---

## Endpoints

### GET /score/{wallet}

Returns the current credit score and risk assessment for a Stellar wallet.

**Request**
```
GET /score/GBZXFMN7AJ4K9PQRST...
Authorization: Bearer sk_live_...
```

**Response**
```json
{
  "wallet": "GBZXFMN7AJ4K9PQRST...",
  "score": 742,
  "risk_tier": "B",
  "loan_eligible": true,
  "max_loan_usdc": 12500,
  "percentile": 81,
  "last_updated": "2026-06-19T14:22:11Z",
  "factors": {
    "payment_history": 96,
    "transaction_volume": 82,
    "account_age": 71,
    "credit_diversity": 64,
    "recent_inquiries": 90
  }
}
```

---

### GET /credential/{wallet}

Returns the latest W3C Verifiable Credential for a wallet. The user must have authorized sharing.

**Response**
```json
{
  "credential_id": "cred-stellar-742-20260619",
  "type": "CreditScoreCredential",
  "issuer": "did:creditrails:protocol",
  "subject": "did:stellar:GBZXFMN7AJ4K9PQRST...",
  "issued_at": "2026-06-19T00:00:00Z",
  "expires_at": "2027-06-19T00:00:00Z",
  "jwt": "eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCJ9..."
}
```

---

### POST /verify

Verifies a credential submitted by a user. Returns verification result without contacting the original wallet.

**Request**
```json
{
  "jwt": "eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCJ9..."
}
```

**Response**
```json
{
  "valid": true,
  "score": 742,
  "risk_tier": "B",
  "loan_eligible": true,
  "issued_at": "2026-06-19T00:00:00Z",
  "expires_at": "2027-06-19T00:00:00Z",
  "issuer_verified": true
}
```

---

## Rate Limits

| Plan | Requests/min | Price |
|---|---|---|
| Free | 10 | $0 |
| Developer | 100 | $49/mo |
| Enterprise | Unlimited | Custom |

---

## Error Codes

| Code | Meaning |
|---|---|
| 401 | Invalid or missing API key |
| 404 | Wallet not found / not indexed yet |
| 422 | Invalid credential JWT |
| 429 | Rate limit exceeded |
