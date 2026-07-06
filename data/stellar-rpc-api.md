# Stellar RPC API Reference

## Endpoints

| Provider | Network  | URL                                      |
|----------|----------|------------------------------------------|
| Official | Testnet  | https://soroban-testnet.stellar.org      |
| LOBSTR   | Mainnet  | https://horizon.stellar.lobstr.co        |

---

## getEvents

Fetch a filtered list of events emitted within a given ledger range.

- Max range: 7 days of recent ledgers
- Default retention: 24 hours
- Deduplicate events by the `id` field when making multiple requests

### Params

| # | Name         | Type             | Notes |
|---|--------------|------------------|-------|
| 1 | `startLedger`| number           | Inclusive. Omit if using `cursor`. |
| 2 | `endLedger`  | number           | Exclusive. Omit if using `cursor`. |
| 3 | `filters`    | array (max 5)    | Each filter matches on `type`, `contractIds` (max 5), `topics` (max 5 filters × 4 segments). |
| 4 | `pagination` | object           | `cursor` (string) and `limit` (1–10000, default 100). |
| 5 | `xdrFormat`  | string           | `"base64"` (default) or `"json"`. |

### Filter `type` values
- `system`
- `contract`

### Result fields

| Field               | Type    | Description |
|---------------------|---------|-------------|
| `latestLedger`      | number  | Latest ledger known to RPC at request time. |
| `events[].type`     | string  | `contract`, `system` |
| `events[].ledger`   | number  | Ledger sequence where event was emitted. |
| `events[].ledgerClosedAt` | string | ISO-8601 timestamp. |
| `events[].contractId` | string | StrKey contract address. |
| `events[].id`       | string  | Unique TOID-based ID (e.g. `0000859036408881152-0000000003`). |
| `events[].txHash`   | string  | 64-char hex transaction hash. |
| `events[].topic`    | string[] | ScVal topics (base64). 1–4 items. |
| `events[].value`    | string  | Emitted data (ScVal, base64). |
| `cursor`            | string  | Paging token for next page. |

### Example — Native XLM Transfer Events

```js
let requestBody = {
  "jsonrpc": "2.0",
  "id": 8675309,
  "method": "getEvents",
  "params": {
    "startLedger": 199616,
    "filters": [
      {
        "type": "contract",
        "contractIds": [
          "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC"
        ],
        "topics": [
          [
            "AAAADwAAAAh0cmFuc2Zlcg==", // "transfer" symbol
            "*",
            "*",
            "**"
          ]
        ]
      }
    ],
    "pagination": { "limit": 2 }
  }
}

let res = await fetch('https://soroban-testnet.stellar.org', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(requestBody),
})
```

---

## getLatestLedger

Returns the latest ledger known to this RPC node.

### Params
None.

### Result fields

| Field         | Type   | Description |
|---------------|--------|-------------|
| `id`          | string | 64-char hex hash of the latest ledger. |
| `sequence`    | number | Sequence number of the latest ledger. |
| `closeTime`   | string | Timestamp of ledger close. |
| `headerXdr`   | string | Base64-encoded `LedgerHeader`. |
| `metadataXdr` | string | Base64-encoded `LedgerCloseMeta`. |

### Example

```js
let requestBody = {
  "jsonrpc": "2.0",
  "id": 8675309,
  "method": "getLatestLedger"
}

let res = await fetch('https://soroban-testnet.stellar.org', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(requestBody),
})
```

---

## getLedgerEntries

Read current values of ledger entries directly (accounts, trustlines, offers, contract data/code, etc.).

- Max 200 keys per request
- Primary way to read contract state not surfaced by events or `simulateTransaction`

### Params

| # | Name        | Type     | Notes |
|---|-------------|----------|-------|
| 1 | `keys`      | string[] | **Required.** Array of base64-encoded `LedgerKey` XDR. Max 200. |
| 2 | `xdrFormat` | string   | `"base64"` (default) or `"json"`. |

### Result fields

| Field                      | Type   | Description |
|----------------------------|--------|-------------|
| `latestLedger`             | number | Latest ledger at request time. |
| `entries[].key`            | string | Base64 `LedgerKey`. |
| `entries[].xdr`            | string | Base64 `LedgerEntryData`. |
| `entries[].lastModifiedLedgerSeq` | number | Ledger when entry last changed. |
| `entries[].liveUntilLedgerSeq`    | number | Ledger until which entry is live (0 = expired). |

### LedgerKey Types

| Type              | Use case |
|-------------------|----------|
| Account           | Full account state (balance, signers, seq num) |
| Trustline         | Non-native asset balance for an account |
| Offer             | DEX offer entry |
| Account Data      | Key-value data attached to account |
| Claimable Balance | Claimable balance entry |
| Liquidity Pool    | Constant-product pool config |
| Contract Data     | Contract storage slot (key → value) |
| Contract Code     | Deployed Wasm bytecode (identified by hash) |
| Config Setting    | Active network configuration |
| TTL               | Time-to-live for contract data/code |

### Building LedgerKeys (TypeScript)

**Account**
```ts
import { Keypair, xdr } from "@stellar/stellar-sdk";

const key = xdr.LedgerKey.ledgerKeyAccount(
  new xdr.LedgerKeyAccount({
    accountId: Keypair.fromPublicKey(publicKey).xdrAccountId(),
  }),
);
```

**Trustline**
```ts
const key = xdr.LedgerKey.ledgerKeyTrustLine(
  new xdr.LedgerKeyTrustLine({
    accountId: Keypair.fromPublicKey(publicKey).xdrAccountId(),
    asset: new Asset("USDC", "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN")
      .toTrustLineXDRObject(),
  }),
);
```

**Contract Data (e.g. COUNTER symbol)**
```ts
import { xdr, Address } from "@stellar/stellar-sdk";

const key = xdr.LedgerKey.contractData(
  new xdr.LedgerKeyContractData({
    contract: new Address(contractId).toScAddress(),
    key: xdr.ScVal.scvSymbol("COUNTER"),
    durability: xdr.ContractDataDurability.persistent(),
    // use xdr.ContractDataDurability.temporary() for temp storage
  }),
);
```

**Contract Code (two-step)**
```ts
import { Contract, xdr } from "@stellar/stellar-sdk";

// Step 1: get the contract instance entry (points to wasm hash)
const instanceKey = new Contract(contractId).getFootprint();

// Step 2: from the entry, extract wasm hash and fetch the code
function getLedgerKeyWasmId(contractData: xdr.ContractDataEntry): xdr.LedgerKey {
  const wasmHash = contractData.val().instance().executable().wasmHash();
  return xdr.LedgerKey.contractCode(
    new xdr.LedgerKeyContractCode({ hash: wasmHash }),
  );
}
```

### Fetching entries

```ts
const s = new Server("https://soroban-testnet.stellar.org");

const response = await s.getLedgerEntries(key1, key2, key3);

const account      = response.entries[0].account();
const trustline    = response.entries[1].trustline();
const contractData = response.entries[2].contractData();
```

### Example request (raw JSON-RPC)

```json
{
  "jsonrpc": "2.0",
  "id": 8675309,
  "method": "getLedgerEntries",
  "params": {
    "keys": [
      "AAAABgAAAAHMA/50/Q+w3Ni8UXWm/trxFBfAfl6De5kFttaMT0/ACwAAABAAAAABAAAAAgAAAA8AAAAHQ291bnRlcgAAAAASAAAAAAAAAAAg4dbAxsGAGICfBG3iT2cKGYQ6hK4sJWzZ6or1C5v6GAAAAAE="
    ]
  }
}
```

### Inspecting XDR via CLI

```bash
echo 'AAAAAAAAAAAL76GC5jcgEGfLG9+nptaB9m+R44oweeN3EcqhstdzhQ==' \
  | stellar xdr decode --type LedgerKey --output json-formatted
```

---

## getLedgers

Returns a paginated list of ledgers starting from `startLedger`.

- Max `limit`: 200 (default 50)
- Retention window: same as the RPC node's history

### Params

| # | Name          | Type   | Notes |
|---|---------------|--------|-------|
| 1 | `startLedger` | number | Inclusive. Omit if using `cursor`. |
| 2 | `pagination`  | object | `cursor` (string) and `limit` (1–200, default 50). |
| 3 | `xdrFormat`   | string | `"base64"` (default) or `"json"`. |

### Result fields

| Field                    | Type     | Description |
|--------------------------|----------|-------------|
| `ledgers[].hash`         | string   | Ledger header hash. |
| `ledgers[].sequence`     | number   | Ledger sequence / block height. |
| `ledgers[].ledgerCloseTime` | string | Unix timestamp of ledger close. |
| `ledgers[].headerXdr`    | string   | Base64 `LedgerHeaderHistoryEntry`. |
| `ledgers[].metadataXdr`  | string   | Base64 `LedgerCloseMeta`. |
| `latestLedger`           | number   | Latest ledger at request time. |
| `latestLedgerCloseTime`  | number   | Unix timestamp of latest ledger close. |
| `oldestLedger`           | number   | Oldest ledger ingested by this node. |
| `oldestLedgerCloseTime`  | number   | Unix timestamp of oldest ledger close. |
| `cursor`                 | string   | Paging token for next page. |

### Example

```js
let requestBody = {
  "jsonrpc": "2.0",
  "id": 8675309,
  "method": "getLedgers",
  "params": {
    "startLedger": 36233,
    "pagination": { "limit": 2 }
  }
}

let res = await fetch('https://soroban-testnet.stellar.org', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(requestBody),
})
```

---

## getNetwork

Returns general information about the configured network (passphrase, protocol version, friendbot URL).

### Params
None.

### Result fields

| Field             | Type   | Required | Description |
|-------------------|--------|----------|-------------|
| `passphrase`      | string | Yes      | Network passphrase. |
| `protocolVersion` | number | Yes      | Current Stellar Core protocol version. |
| `friendbotUrl`    | string | No       | Faucet URL (testnet only). |

### Example

```js
let requestBody = {
  "jsonrpc": "2.0",
  "id": 8675309,
  "method": "getNetwork"
}

// Testnet result:
// {
//   "friendbotUrl": "https://friendbot-testnet.stellar.org/",
//   "passphrase": "Test SDF Network ; September 2015",
//   "protocolVersion": 20
// }
```

---

## getTransaction

Returns details about a specific transaction. Poll this after submitting a transaction to confirm inclusion.

- Default retention: 24 hours (configurable up to 7 days for private nodes)
- For history beyond 7 days: index yourself or use Hubble (BigQuery)

### Params

| # | Name        | Type   | Notes |
|---|-------------|--------|-------|
| 1 | `hash`      | string | **Required.** 64-char hex transaction hash. |
| 2 | `xdrFormat` | string | `"base64"` (default) or `"json"`. |

### Result fields

| Field                   | Type     | Always present | Description |
|-------------------------|----------|----------------|-------------|
| `status`                | string   | Yes | `SUCCESS`, `NOT_FOUND`, or `FAILED`. |
| `latestLedger`          | number   | Yes | Latest ledger at request time. |
| `latestLedgerCloseTime` | number   | Yes | Unix timestamp of latest ledger. |
| `oldestLedger`          | number   | Yes | Oldest ledger in node. |
| `oldestLedgerCloseTime` | number   | Yes | Unix timestamp of oldest ledger. |
| `ledger`                | number   | If SUCCESS/FAILED | Ledger that included the tx. |
| `createdAt`             | number   | If SUCCESS/FAILED | Unix timestamp of inclusion. |
| `applicationOrder`      | number   | If SUCCESS/FAILED | Index of tx within the ledger. |
| `feeBump`               | boolean  | If SUCCESS/FAILED | Whether the tx was fee bumped. |
| `envelopeXdr`           | string   | No | Base64 `TransactionEnvelope`. |
| `resultXdr`             | string   | If SUCCESS/FAILED | Base64 `TransactionResult`. |
| `resultMetaXdr`         | string   | No | Base64 `TransactionMeta`. |
| `diagnosticEventsXdr`   | string[] | No | Base64 `DiagnosticEvent` slice (requires `ENABLE_SOROBAN_DIAGNOSTIC_EVENTS` on RPC). |
| `events.transactionEventsXdr` | string[] | No | Fee charge/refund events. |
| `events.contractEventsXdr`    | string[][] | No | Contract events per operation. |

### Example

```js
let requestBody = {
  "jsonrpc": "2.0",
  "id": 8675309,
  "method": "getTransaction",
  "params": {
    "hash": "32f7e5c3afd281fcaa99c0e990adf62f33e3bb341b1641a5c8b0b4a4dc55c487"
  }
}

let res = await fetch('https://soroban-testnet.stellar.org', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(requestBody),
})
```

### SDK usage

```python
from stellar_sdk import SorobanServer

server = SorobanServer('https://soroban-testnet.stellar.org')
tx = server.get_transaction("6bc97bddc21811c626839baf4ab574f4f9f7ddbebb44d286ae504396d4e752da")
print(tx.status)  # SUCCESS / NOT_FOUND / FAILED
```

---

## getVersionInfo

Returns version details about the RPC server and its embedded Captive Core instance.

### Params
None.

### Result fields

| Field                | Type    | Example |
|----------------------|---------|---------|
| `version`            | string  | `"23.0.1"` |
| `commit_hash`        | string  | `"fcd2f0523f04279bae4502f3e3fa00ca627e6f6a"` |
| `build_time_stamp`   | string  | `"2025-05-10T11:18:38"` |
| `captive_core_version` | string | `"stellar-core 23.0.1 (050eacf11a15afb2e95560dfb5723dfdcf78070f)"` |
| `protocol_version`   | integer | `23` |

### Example

```js
let requestBody = {
  "jsonrpc": "2.0",
  "id": 8675309,
  "method": "getVersionInfo"
}

// Result:
// {
//   "version": "23.0.1",
//   "commit_hash": "fcd2f0523f04279bae4502f3e3fa00ca627e6f6a",
//   "build_time_stamp": "2025-05-10T11:18:38",
//   "captive_core_version": "stellar-core 23.0.1 (...)",
//   "protocol_version": 23
// }
```

---

## sendTransaction

Submit a signed transaction to the Stellar network. **This is the only way to make changes on-chain.**

- Does NOT wait for confirmation — returns immediately after enqueue
- Follow up with `getTransaction` to poll for final status
- Supports all transaction types (not just smart contracts)

### Params

| # | Name          | Type   | Notes |
|---|---------------|--------|-------|
| 1 | `transaction` | string | **Required.** Base64-encoded signed `TransactionEnvelope`. |

### Result fields

| Field                   | Type     | Required | Description |
|-------------------------|----------|----------|-------------|
| `hash`                  | string   | Yes | 64-char hex transaction hash. |
| `status`                | string   | Yes | `PENDING`, `DUPLICATE`, `TRY_AGAIN_LATER`, or `ERROR`. |
| `latestLedger`          | number   | Yes | Latest ledger at request time. |
| `latestLedgerCloseTime` | number   | Yes | Unix timestamp of latest ledger close. |
| `errorResultXdr`        | string   | If ERROR | Base64 `TransactionResult` with rejection details. |
| `diagnosticEventsXdr`   | string[] | If ERROR | Base64 `DiagnosticEvent` array with rejection details. |

### Example

```js
let requestBody = {
  "jsonrpc": "2.0",
  "id": 8675309,
  "method": "sendTransaction",
  "params": {
    "transaction": "<base64-signed-envelope>"
  }
}

// On success:
// { "status": "PENDING", "hash": "d8ec9b68...", "latestLedger": 2553978, ... }
```

### SDK usage (Python)

```python
from stellar_sdk import SorobanServer, Keypair, Network, TransactionBuilder, scval

server = SorobanServer('https://soroban-testnet.stellar.org')
root_keypair = Keypair.from_secret("SXXX...")
root_account = server.load_account(root_keypair.public_key)

contract_id = "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC"
tx = (
    TransactionBuilder(root_account, Network.TESTNET_NETWORK_PASSPHRASE, base_fee=100)
    .append_invoke_contract_function_op(contract_id, "transfer", [
        scval.to_address(root_keypair.public_key),
        scval.to_address("GBRPYHIL2CI3FNQ4BXLFMNDLFJUNPU2HY3ZMFSHONUCEOASW7QC7OX2H"),
        scval.to_int128(1 * 10**7),  # 1 XLM (7 decimal places)
    ])
    .set_timeout(30)
    .build()
)
tx = server.prepare_transaction(tx)
tx.sign(root_keypair)
response = server.send_transaction(tx)
print(response.status, response.hash)
```

---

## simulateTransaction

Dry-run a contract invocation to get the effective transaction data, required authorizations, and minimum resource fee — without submitting anything on-chain.

- Can also invoke **read-only** contract functions for free
- Transaction must contain exactly **one** `invokeHostFunction` operation

### Params

| # | Name              | Type   | Notes |
|---|-------------------|--------|-------|
| 1 | `transaction`     | string | **Required.** Base64 `TransactionEnvelope` with a single `invokeHostFunction` op. |
| 2 | `resourceConfig`  | object | Optional. `instructionLeeway` (number) — extra instruction budget. |
| 3 | `xdrFormat`       | string | `"base64"` (default) or `"json"`. |
| 4 | `authMode`        | string | `"enforce"` (default), `"record"`, or `"record_allow_nonroot"`. |

### Result fields

| Field                         | Type      | Notes |
|-------------------------------|-----------|-------|
| `latestLedger`                | number    | Latest ledger at request time. |
| `minResourceFee`              | string    | Stringified number — add to Stellar network fee before submitting. |
| `transactionData`             | string    | Base64 `SorobanTransactionData` — use when building the real tx. |
| `results[].xdr`               | string    | Return value of the host function call (base64). |
| `results[].auth`              | string[]  | Per-address authorizations recorded (base64 array). |
| `events`                      | string[]  | Events emitted during simulation (base64, ordered by emission). |
| `restorePreamble.minResourceFee` | string | Present if archived entries need `RestoreFootprint` first. |
| `restorePreamble.transactionData` | string | `SorobanTransactionData` for the `RestoreFootprint` tx. |
| `stateChanges[].type`         | string    | `created`, `updated`, or `deleted`. |
| `stateChanges[].key`          | string    | Base64 `LedgerKey`. |
| `stateChanges[].before`       | string\|null | Base64 `LedgerEntry` before simulation (null on creation). |
| `stateChanges[].after`        | string\|null | Base64 `LedgerEntry` after simulation (null on deletion). |
| `error`                       | string    | Present if simulation failed — explains why. |

### Example

```js
let requestBody = {
  "jsonrpc": "2.0",
  "id": 8675309,
  "method": "simulateTransaction",
  "params": {
    "transaction": "<base64-envelope-with-invokeHostFunction>",
    "resourceConfig": {
      "instructionLeeway": 3000000
    }
  }
}

// On success result includes:
// transactionData   → use as SorobanTransactionData when building the real tx
// minResourceFee    → add to your base fee
// results[0].xdr   → return value of the function
// results[0].auth  → required authorizations
```

---

## Method Summary

| Method                | Purpose |
|-----------------------|---------|
| `getEvents`           | Filtered event log by ledger range / contract / topic |
| `getLatestLedger`     | Current ledger sequence, hash, close time |
| `getLedgerEntries`    | Live ledger state (accounts, trustlines, contract data/code) |
| `getLedgers`          | Paginated ledger list with full XDR |
| `getNetwork`          | Network passphrase, protocol version, friendbot URL |
| `getTransaction`      | Status and details of a submitted transaction |
| `getVersionInfo`      | RPC server + Captive Core version info |
| `sendTransaction`     | Submit a signed transaction (fire-and-forget) |
| `simulateTransaction` | Dry-run a contract invocation, get fees + auth |
