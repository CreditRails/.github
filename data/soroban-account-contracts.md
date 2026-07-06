# Soroban Account Contracts

Source: https://github.com/stellar/soroban-examples/tree/v23.0.0

> **Warning**: Contract accounts require a thorough understanding of auth/signing. These examples are API references, not production-ready code.

---

## Simple Account

Single ed25519 key. Every `require_auth` call on this contract address delegates to that key.

### Storage key

```rust
#[contracttype]
#[derive(Clone)]
pub enum DataKey {
    Owner,
}
```

### Initialize (store owner public key)

```rust
pub fn init(env: Env, public_key: BytesN<32>) {
    if env.storage().instance().has(&DataKey::Owner) {
        panic!("owner is already set");
    }
    env.storage().instance().set(&DataKey::Owner, &public_key);
}
```

### `__check_auth` (called by host on every `require_auth`)

```rust
#[allow(non_snake_case)]
pub fn __check_auth(
    env: Env,
    signature_payload: BytesN<32>,
    signature: BytesN<64>,
    _auth_context: Vec<Context>,
) {
    let public_key: BytesN<32> = env
        .storage()
        .instance()
        .get(&DataKey::Owner)
        .unwrap();
    env.crypto()
        .ed25519_verify(&public_key, &signature_payload.into(), &signature);
}
```

- `__check_auth` is reserved — can only be invoked by the Soroban host during `require_auth`
- Panics on invalid signature → upstream `require_auth` rejects

### Testing `__check_auth`

Use `env.try_invoke_contract_check_auth` (can't call `__check_auth` directly):

```rust
#[test]
fn test_account() {
    let env = Env::default();
    let account_contract = SimpleAccountClient::new(&env, &env.register(SimpleAccount, ()));

    let signer = Keypair::generate(&mut thread_rng());
    account_contract.init(&signer.public.to_bytes().into_val(&env));

    let payload = BytesN::random(&env);

    // Valid signature — should succeed
    env.try_invoke_contract_check_auth::<Error>(
        &account_contract.address,
        &payload,
        sign(&env, &signer, &payload),
        &vec![&env],
    )
    .unwrap();

    // Random bytes — should fail
    assert!(env
        .try_invoke_contract_check_auth::<Error>(
            &account_contract.address,
            &payload,
            BytesN::<64>::random(&env).into(),
            &vec![&env],
        )
        .is_err());
}
```

---

## Complex Account (Multisig + Spend Limits)

Multiple ed25519 keys with equal weight, plus per-token spend limits for partial-signing scenarios.

### Storage keys

```rust
#[contracttype]
#[derive(Clone)]
enum DataKey {
    SignerCnt,
    Signer(BytesN<32>),
    SpendLimit(Address),
}
```

### Constructor (Protocol 23 — atomic init)

```rust
pub fn __constructor(env: Env, signers: Vec<BytesN<32>>) {
    for signer in signers.iter() {
        env.storage().instance().set(&DataKey::Signer(signer), &());
    }
    env.storage().instance().set(&DataKey::SignerCnt, &signers.len());
}
```

Using `__constructor` ensures the contract is created and initialized atomically, preventing front-running attacks where an attacker could initialize with their own keys.

### Set per-token spend limit

```rust
pub fn add_limit(env: Env, token: Address, limit: i128) {
    env.current_contract_address().require_auth();
    env.storage().instance().set(&DataKey::SpendLimit(token), &limit);
}
```

### Signature type

```rust
#[contracttype]
#[derive(Clone)]
pub struct AccSignature {
    pub public_key: BytesN<32>,
    pub signature: BytesN<64>,
}
```

### Error codes

```rust
#[contracterror]
#[derive(Copy, Clone, Debug, Eq, PartialEq, PartialOrd, Ord)]
#[repr(u32)]
pub enum AccError {
    NotEnoughSigners = 1,
    NegativeAmount   = 2,
    BadSignatureOrder = 3,
    UnknownSigner    = 4,
}
```

### `__check_auth` flow

```rust
fn __check_auth(
    env: Env,
    signature_payload: Hash<32>,
    signatures: Vec<AccSignature>,
    auth_context: Vec<Context>,
) -> Result<(), AccError> {
    authenticate(&env, &signature_payload, &signatures)?;

    let tot_signers: u32 = env.storage().instance().get(&DataKey::SignerCnt).unwrap();
    let all_signed = tot_signers == signatures.len();

    let curr_contract = env.current_contract_address();
    let mut spend_left_per_token = Map::<Address, i128>::new(&env);

    for context in auth_context.iter() {
        verify_authorization_policy(&env, &context, &curr_contract, all_signed, &mut spend_left_per_token)?;
    }
    Ok(())
}
```

**Authentication** — signatures must be:
- Ordered by public key (ascending) — prevents duplicates
- Each signer must be registered in storage
- `ed25519_verify` runs on each

**Authorization policy** — without all signers:
- Cannot modify the account contract itself
- Cannot create new contracts
- Token `transfer`/`approve`/`burn` is checked against per-token spend limits

### Testing policy

```rust
// Sign with one signer (limit of 1000)
env.try_invoke_contract_check_auth::<AccError>(
    &account_contract.address,
    &payload,
    vec![&env, sign(&env, &signers[0], &payload)].into(),
    &vec![&env, token_auth_context(&env, &token, Symbol::new(&env, "transfer"), 1000)],
).unwrap();

// Exceeds limit — should fail with NotEnoughSigners
assert_eq!(
    env.try_invoke_contract_check_auth::<AccError>(
        &account_contract.address,
        &payload,
        vec![&env, sign(&env, &signers[0], &payload)].into(),
        &vec![&env, token_auth_context(&env, &token, Symbol::new(&env, "transfer"), 1001)],
    ).err().unwrap().unwrap(),
    AccError::NotEnoughSigners
);
```

### Build & run

```bash
cargo test -p soroban-account-contract
stellar contract build
```
