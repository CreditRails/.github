# Stellar Info File (SEP-1)

## Overview

SEP-1 allows wallets and other Stellar applications to discover information about your anchor service by hosting a `stellar.toml` file at `/.well-known/stellar.toml`.

Discoverable via SEP-1:
- Organization information
- Supported assets and currencies
- Authentication endpoints (SEP-10)
- SEP endpoints for SEP-6, SEP-24, SEP-31, SEP-38, SEP-45

Reference: [SEP-0001 specification](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0001.md)

---

## Minimal stellar.toml

```toml
# dev.stellar.toml
ACCOUNTS = ["GD...G"]  # Your distribution account public keys
SIGNING_KEY = "GD...G"  # Your signing key (public key) for SEP-10 authentication
NETWORK_PASSPHRASE = "Test SDF Network ; September 2015"  # Use "Public Global Stellar Network ; September 2015" for mainnet

[DOCUMENTATION]
ORG_NAME = "Your organization"
ORG_URL = "https://your-website.com"
ORG_DESCRIPTION = "A description of your organization"
```

Additional sections needed as you add features: `[[CURRENCIES]]`, `TRANSFER_SERVER`, `TRANSFER_SERVER_SEP0024`, `WEB_AUTH_ENDPOINT`, `WEB_AUTH_FOR_CONTRACTS_ENDPOINT`, `DIRECT_PAYMENT_SERVER`.

**Production vs. Development**: Use separate stellar.toml files for testnet and mainnet with the respective `NETWORK_PASSPHRASE`.

---

## Configuration

The Anchor Platform supports three methods to serve the file:

| Type | Use Case | Description |
|------|----------|-------------|
| `file` | Recommended for most cases | Read from a local file on the server |
| `string` | Quick testing or simple configs | Provide the TOML content directly in the config |
| `url` | External hosting | Fetch from a remote URL |

### Environment Variables

```bash
SEP1_ENABLED=true
SEP1_TOML_TYPE=file       # one of: file, string, url
SEP1_TOML_VALUE=/path/to/your/stellar.toml
```

### Method 1: File (Recommended)

```bash
SEP1_ENABLED=true
SEP1_TOML_TYPE=file
SEP1_TOML_VALUE=/path/to/your/stellar.toml
```

Docker volume mount:
```yaml
# docker-compose.yaml
volumes:
  - ./config/stellar.toml:/config/stellar.toml:ro
```

### Method 2: String

```bash
SEP1_ENABLED=true
SEP1_TOML_TYPE=string
SEP1_TOML_VALUE="ACCOUNTS = [\"GD...G\"]
SIGNING_KEY = \"GD...G\"
NETWORK_PASSPHRASE = \"Test SDF Network ; September 2015\"
[DOCUMENTATION]
ORG_NAME = \"Your organization\"
ORG_URL = \"https://your-website.com\""
```

### Method 3: URL

```bash
SEP1_ENABLED=true
SEP1_TOML_TYPE=url
SEP1_TOML_VALUE=https://example.com/stellar.toml
```

The Anchor Platform fetches the file on each request when using `url` type.

---

## Endpoints

Once configured, the platform serves:
- `/.well-known/stellar.toml` — primary endpoint (`Content-Type: text/plain`)
- `/` — redirects to `/.well-known/stellar.toml` when SEP-1 is enabled

### Test

```bash
curl http://localhost:8080/.well-known/stellar.toml
curl -L http://localhost:8080/
```

---

## External Hosting Alternative

Host the file at `https://your-domain.com/.well-known/stellar.toml` via nginx or CDN. If doing so, the Anchor Platform's SEP-1 service is optional — but it's convenient to manage everything in one place.
