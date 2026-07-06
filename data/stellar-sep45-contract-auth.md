# Stellar Authentication for Contract Accounts (SEP-45)

## Overview

SEP-45 (Stellar Web Authentication for Contract Accounts) enables smart wallet applications to create authenticated sessions by proving control over a **contract account** (`C...`). The server issues a JWT used in subsequent requests.

Reference: [SEP-0045 specification](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0045.md)

**SEP-45 vs SEP-10**: SEP-45 supports contract accounts (`C...`); SEP-10 supports classic (`G...`) and muxed (`M...`) accounts. To support all account types, implement both.

Supported features:
- Challenge/Response Flow: `GET /sep45/auth` → authorization entries, `POST /sep45/auth` → JWT
- Contract Account Authentication using Soroban authorization entries
- Transaction Simulation: automatic simulation to verify authorization entries
- Multiple Home Domains and wildcard patterns

A reference SEP-45 contract is deployed at `CD3LA6RKF5D2FN2R2L57MWXLBRSEWWENE74YBEFZSSGNJRJGICFGQXMX` on testnet.

---

## Authentication Flow

1. Client requests a unique challenge (`GET /sep45/auth`)
2. Client verifies and signs the challenge
3. Client submits the signed challenge (`POST /sep45/auth`)
4. Server verifies and responds with a JWT session token

---

## Enable SEP-45

SEP-45 requires Stellar RPC to simulate transactions.

```bash
# dev.env
STELLAR_NETWORK_TYPE=rpc
STELLAR_NETWORK_RPC_URL=https://soroban-testnet.stellar.org
SEP45_ENABLED=true
SEP45_HOME_DOMAINS=localhost:8080
SEP45_WEB_AUTH_CONTRACT_ID="CD3LA6RKF5D2FN2R2L57MWXLBRSEWWENE74YBEFZSSGNJRJGICFGQXMX"
SECRET_SEP10_SIGNING_SEED="a Stellar private key"
SECRET_SEP45_JWT_SECRET="a secret encryption key"
```

### Required Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `STELLAR_NETWORK_TYPE` | Required | Must be `rpc` for SEP-45. |
| `STELLAR_NETWORK_RPC_URL` | Required | URL of the Stellar RPC server for transaction simulation. |
| `SEP45_ENABLED` | `false` | Set to `true` to enable SEP-45. |
| `SEP45_HOME_DOMAINS` | Required | Comma-separated home domains. Supports wildcards. Must match where `stellar.toml` is served. |
| `SEP45_WEB_AUTH_CONTRACT_ID` | Required | Contract ID implementing `web_auth_verify`. Must match `WEB_AUTH_CONTRACT_ID` in stellar.toml. |
| `SECRET_SEP10_SIGNING_SEED` | Required | Private key for signing challenges (shared with SEP-10). |
| `SECRET_SEP45_JWT_SECRET` | Required | Key used to sign and verify issued JWTs. |

### Optional Variables

```bash
SEP45_WEB_AUTH_DOMAIN=localhost:8080   # default: first home_domain if only one
SEP45_AUTH_TIMEOUT=900                 # challenge validity in seconds
SEP45_JWT_TIMEOUT=86400                # JWT validity in seconds (24h)
```

| Variable | Default | Description |
|----------|---------|-------------|
| `SEP45_WEB_AUTH_DOMAIN` | First home_domain (if only one) | Required if multiple home_domains or wildcards. Must match host of the SEP server. |
| `SEP45_AUTH_TIMEOUT` | `900` | Seconds a challenge remains valid. |
| `SEP45_JWT_TIMEOUT` | `86400` | Seconds a JWT remains valid. |

---

## stellar.toml Update

```toml
ACCOUNTS = ["add your public keys for distribution accounts here"]
SIGNING_KEY = "add your signing key here (public key from SECRET_SEP10_SIGNING_SEED)"
WEB_AUTH_FOR_CONTRACTS_ENDPOINT = "http://localhost:8080/sep45/auth"
WEB_AUTH_CONTRACT_ID = "CD3LA6RKF5D2FN2R2L57MWXLBRSEWWENE74YBEFZSSGNJRJGICFGQXMX"
```

- `SIGNING_KEY`: public key from `SECRET_SEP10_SIGNING_SEED`
- `WEB_AUTH_FOR_CONTRACTS_ENDPOINT`: use `https://` in production; host must match `SEP45_HOME_DOMAINS`
- `WEB_AUTH_CONTRACT_ID`: must match `SEP45_WEB_AUTH_CONTRACT_ID`

Endpoint must support:
- `GET <WEB_AUTH_FOR_CONTRACTS_ENDPOINT>` — request authorization entries
- `POST <WEB_AUTH_FOR_CONTRACTS_ENDPOINT>` — exchange signed entries for JWT
