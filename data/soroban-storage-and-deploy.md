# Soroban: Storing Data & Deploying Contracts

## Storing Data (Increment Contract)

Source: https://github.com/stellar/soroban-examples/tree/v23.0.0/increment

### Contract Data Keys

```rust
const COUNTER: Symbol = symbol_short!("COUNTER");
```

- `Symbol`: short string type (up to 32 chars, only `a-zA-Z0-9_`)
- `symbol_short!()`: compile-time constant for symbols ≤ 9 characters
- If symbol > 9 chars, use `Symbol::new(&env, "longer_key")` at runtime

### Storage API

```rust
// Read (with default fallback)
let mut count: u32 = env.storage().instance().get(&COUNTER).unwrap_or(0);

// Write
env.storage().instance().set(&COUNTER, &count);

// Extend TTL to prevent archival
env.storage().instance().extend_ttl(50, 100);
// arg1: threshold — if remaining TTL < this, extend
// arg2: extend_to — new TTL in ledgers from now
```

### Storage Types

| Type         | Use case |
|--------------|----------|
| `instance()` | Contract-level data; shared TTL with the contract instance |
| `persistent()` | Large or per-user data; each entry has its own TTL |
| `temporary()` | Short-lived data; NOT automatically restored after archival |

### Full Increment Contract

```rust
#![no_std]
use soroban_sdk::{contract, contractimpl, log, symbol_short, Env, Symbol};

const COUNTER: Symbol = symbol_short!("COUNTER");

#[contract]
pub struct IncrementContract;

#[contractimpl]
impl IncrementContract {
    pub fn increment(env: Env) -> u32 {
        let mut count: u32 = env.storage().instance().get(&COUNTER).unwrap_or(0);
        log!(&env, "count: {}", count);
        count += 1;
        env.storage().instance().set(&COUNTER, &count);
        env.storage().instance().extend_ttl(50, 100);
        count
    }
}

mod test;
```

### Tests (test.rs)

```rust
#![cfg(test)]
use crate::{IncrementContract, IncrementContractClient};
use soroban_sdk::Env;

#[test]
fn test() {
    let env = Env::default();
    let contract_id = env.register(IncrementContract, ());
    let client = IncrementContractClient::new(&env, &contract_id);

    assert_eq!(client.increment(), 1);
    assert_eq!(client.increment(), 2);
    assert_eq!(client.increment(), 3);
}
```

Run with logs: `cargo test -- --nocapture`

---

## Events (Protocol 23 / Whisk)

### Defining and publishing an event

```rust
// Define static topics + data format
#[contractevent(topics = ["COUNTER", "increment"], data_format = "single-value")]
struct IncrementEvent {
    count: u32,
}

// Publish inside the contract function
IncrementEvent { count }.publish(&env);
```

**Topic name limits**: topic strings must fit within the 9-char `symbol_short!` limit.

**`data_format` options**:
- `"single-value"` — payload is the field value directly
- `"vec"` — payload wrapped in a vector
- default — follows the struct layout

### Dynamic topics (placed inside the struct with `#[topic]`)

```rust
#[contractevent]
pub struct Increment {
    #[topic]
    addr: Address,  // becomes second topic (struct name "increment" is first)
    count: u32,
}
```

### Testing events

```rust
assert_eq!(
    env.events().all(),
    vec![
        &env,
        (
            contract_id.clone(),
            (symbol_short!("COUNTER"), symbol_short!("increment")).into_val(&env),
            1u32.into_val(&env),
        ),
    ]
);
```

> Events are discarded if the contract invocation fails (panic, budget exhaustion, error return).

---

## Deploying Contracts (Two-Step)

### Step 1 — Upload Wasm (install)

```bash
stellar contract upload \
  --network testnet \
  --source-account alice \
  --wasm target/wasm32v1-none/release/increment.wasm
# returns: <wasm-hash>
```

### Step 2 — Instantiate

```bash
stellar contract deploy \
  --wasm-hash <wasm-hash> \
  --source-account alice \
  --network testnet \
  --alias increment
# returns: <contract-id>
```

### Invoke

```bash
stellar contract invoke \
  --id <contract-id> \
  --source-account alice \
  --network testnet \
  -- \
  increment
```

### One-step shorthand (for dev)

```bash
stellar contract deploy \
  --wasm target/wasm32v1-none/release/soroban_increment_contract.wasm \
  --alias increment_example \
  --source-account alice \
  --network testnet
```

### Build

```bash
stellar contract build
ls target/wasm32v1-none/release/*.wasm
```
