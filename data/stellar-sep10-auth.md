# Stellar Authentication (SEP-10)

## Overview

SEP-10 (Stellar Web Authentication) enables wallet applications to create authenticated sessions with Stellar anchors by proving control over a Stellar account. The server issues a JWT used in subsequent requests.

Reference: [SEP-0010 specification](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0010.md)

Supported features:
- Challenge/Response Flow: `GET /auth` → challenge, `POST /auth` → JWT
- Client Attribution: optional verification of client application identity
- Custodial Wallet Support
- Multiple Home Domains: multiple domains and wildcard patterns

---

## Authentication Flow

1. Client requests a unique challenge from the Server (`GET /auth`)
2. Client verifies and signs the challenge
3. Client submits the signed challenge (`POST /auth`)
4. Server verifies and responds with a JWT session token

---

## Enable SEP-10

```bash
# dev.env
SEP10_ENABLED=true
SEP10_HOME_DOMAINS=localhost:8080
SECRET_SEP10_SIGNING_SEED="a Stellar private key"
SECRET_SEP10_JWT_SECRET="a secret encryption key"
```

### Required Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SEP10_ENABLED` | `false` | Set to `true` to enable SEP-10 |
| `SEP10_HOME_DOMAINS` | `localhost:8080` | Comma-separated home domains. Supports wildcards like `*.stellar.org`. Must match where `stellar.toml` is served. |
| `SECRET_SEP10_SIGNING_SEED` | Required | Private key corresponding to `SIGNING_KEY` in stellar.toml. Signs authentication challenges. |
| `SECRET_SEP10_JWT_SECRET` | Required | Key used to sign and verify issued JWTs. |

### Optional Variables

```bash
SEP10_WEB_AUTH_DOMAIN=localhost:8080      # default: first home_domain if only one
SEP10_AUTH_TIMEOUT=900                    # challenge validity in seconds
SEP10_JWT_TIMEOUT=86400                   # JWT validity in seconds (24h)
SEP10_REQUIRE_AUTH_HEADER=false           # require Bearer JWT on GET /auth
SEP10_CLIENT_ATTRIBUTION_REQUIRED=false   # require client_domain in challenge requests
SEP10_CLIENT_ALLOW_LIST=client1,client2   # restrict to specific clients
```

| Variable | Default | Description |
|----------|---------|-------------|
| `SEP10_WEB_AUTH_DOMAIN` | First home_domain (if only one) | Required if multiple home_domains or wildcard patterns. Must match host of the SEP server. |
| `SEP10_AUTH_TIMEOUT` | `900` | Seconds a challenge transaction remains valid. |
| `SEP10_JWT_TIMEOUT` | `86400` | Seconds a JWT remains valid. |
| `SEP10_REQUIRE_AUTH_HEADER` | `false` | If true, requires `Authorization: Bearer <jwt>` on `GET /auth`. Useful for re-authentication flows. |
| `SEP10_CLIENT_ATTRIBUTION_REQUIRED` | `false` | If true, noncustodial wallets must provide `client_domain`. |
| `SEP10_CLIENT_ALLOW_LIST` | Empty (all allowed) | Comma-separated list of allowed client names. Only relevant when `SEP10_CLIENT_ATTRIBUTION_REQUIRED=true`. |

---

## Client Attribution

Restrict authentication to specific wallet applications and verify their identity. Only needed if you want to restrict access or track wallet applications.

### Enable

```bash
SEP10_CLIENT_ATTRIBUTION_REQUIRED=true
```

When enabled, noncustodial wallets must:
1. Provide `client_domain` in the challenge request
2. Sign the challenge with the `SIGNING_KEY` from that domain's `stellar.toml`
3. Have their domain listed in your client configuration

### Client Configuration (YAML)

```yaml
clients:
  # Custodial client
  - name: bluecorp
    type: custodial
    signing_keys: "signing key 1 of bluecorp","signing key 2 of bluecorp"
    callback_urls:
      sep6: https://callback.bluecorp.com/api/v1/anchor/callback/sep6
      sep12: https://callback.bluecorp.com/api/v1/anchor/callback/sep12
    allow_any_destination: false
    destination_accounts: GA...

  # Noncustodial client
  - name: pinkcorp
    type: noncustodial
    domains: pinkcorp.com
    callback_urls:
      sep6: https://callback.pinkcorp.com/api/v2/anchor/callback/sep6
      sep12: https://callback.pinkcorp.com/api/v2/anchor/callback/sep12

  - name: redcorp
    type: custodial
    signing_keys: "signing key of redcorp"
```

### Client Configuration (Environment Variables)

```bash
CLIENTS[0]_NAME=bluecorp
CLIENTS[0]_TYPE=custodial
CLIENTS[0]_SIGNING_KEYS="signing key 1 of bluecorp","signing key 2 of bluecorp"
CLIENTS[0]_ALLOW_ANY_DESTINATION=false
CLIENTS[0]_DESTINATION_ACCOUNTS=GA...

CLIENTS[1]_NAME=pinkcorp
CLIENTS[1]_TYPE=noncustodial
CLIENTS[1]_DOMAINS=pinkcorp.com

CLIENTS[2]_NAME=redcorp
CLIENTS[2]_TYPE=custodial
CLIENTS[2]_SIGNING_KEYS="signing key of redcorp"
```

---

## stellar.toml Update

```toml
SIGNING_KEY = "your signing key public key"
WEB_AUTH_ENDPOINT = "http://localhost:8080/auth"
```

- `SIGNING_KEY`: public key from `SECRET_SEP10_SIGNING_SEED`
- `WEB_AUTH_ENDPOINT`: must use `https://` in production; host must match `SEP10_HOME_DOMAINS`

Endpoint must support:
- `GET <WEB_AUTH_ENDPOINT>` — request a challenge
- `POST <WEB_AUTH_ENDPOINT>` — exchange signed challenge for JWT

---

## Testing the Flow

```bash
# Get account ID and secret
IDENTITY_NAME="alice"
ACCOUNT_ID=$(stellar keys public-key "$IDENTITY_NAME")
SECRET_SEED=$(stellar keys secret "$IDENTITY_NAME")

# Step 1: Request challenge
CHALLENGE_RESPONSE=$(curl -s "http://localhost:8080/auth?account=$ACCOUNT_ID")
CHALLENGE_XDR=$(echo "$CHALLENGE_RESPONSE" | jq -r '.transaction')

# Step 2: Sign challenge with Stellar CLI
SIGNED_CHALLENGE_XDR=$(echo "$CHALLENGE_XDR" | stellar tx sign --sign-with-key "$SECRET_SEED" 2>&1 | tail -1)

# Step 3: Submit and receive JWT
curl -X POST "http://localhost:8080/auth" \
  -H "Content-Type: application/json" \
  -d "{\"transaction\": \"$SIGNED_CHALLENGE_XDR\"}"
```

Use the returned JWT as `Authorization: Bearer <token>` in subsequent requests.
