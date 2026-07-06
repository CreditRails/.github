# Stellar RPC Providers

## Infrastructure Providers

Providers supporting Futurenet, Testnet, and Mainnet:

| Provider | Futurenet | Testnet | Mainnet | Dedicated Nodes | RPC Archive |
|----------|-----------|---------|---------|-----------------|-------------|
| Blockdaemon | ❌ | ✅ | ✅ | ✅ | ❌ |
| Validation Cloud | ❌ | ✅ | ✅ | ❌ | ❌ |
| QuickNode | ❌ | ✅ | ✅ | ✅ | ❌ |
| NowNodes | ✅ | ✅ | ✅ | ✅ | ❌ |
| Gateway* | ❌ | ✅ | ✅ | ✅ | ✅ |
| Ankr | ❌ | ✅ | ✅ | ❌ | ✅ |
| Infstones | ❌ | ❌ | ✅ | ✅ | ❌ |
| Obsrvr* | ❌ | ✅ | ✅ | ❌ | ✅ |
| Nodies | ❌ | ✅ | ✅ | ❌ | ❌ |
| OnFinality* | ❌ | ❌ | ✅ | ✅ | ✅ |
| Lightsail Network - Quasar* | ❌ | ❌ | ✅ | ❌ | ✅ |
| Uniblock | ❌ | ✅ | ✅ | ❌ | ❌ |
| Exaion | ❌ | ❌ | ✅ | ✅ | ✅ |
| Alchemy | ❌ | ✅ | ✅ | ✅ | ❌ |
| GetBlock | ❌ | ❌ | ✅ | ✅ | ✅ |

\* RPC Archive supports full ledger history via `getLedgers`. Only available from marked providers.

**Dedicated Nodes** = providers who host full nodes as a service.

---

## Publicly Accessible APIs

| Provider | Network | URL |
|----------|---------|-----|
| Liquify | Futurenet | `https://stellar.liquify.com/api=41EEWAH79Y5OCGI7/futurenet` |
| Liquify | Testnet | `https://stellar.liquify.com/api=41EEWAH79Y5OCGI7/testnet` |
| Liquify | Mainnet | `https://stellar-mainnet.liquify.com/api=41EEWAH79Y5OCGI7/mainnet` |
| Gateway | Testnet | `https://soroban-rpc.testnet.stellar.gateway.fm` |
| Gateway | Mainnet | `https://soroban-rpc.mainnet.stellar.gateway.fm` |
| sorobanrpc.com | Mainnet | `https://mainnet.sorobanrpc.com` |
| Nodies | Testnet | `https://stellar-soroban-testnet-public.nodies.app` |
| Nodies | Mainnet | `https://stellar-soroban-public.nodies.app` |
| SDF | Futurenet | `https://rpc-futurenet.stellar.org` |
| SDF | Testnet | `https://soroban-testnet.stellar.org` |
| OnFinality | Mainnet | `https://stellar.api.onfinality.io/public` |
| Lightsail Network - Quasar | Mainnet | `https://rpc.lightsail.network/` |
| Lightsail Network - Quasar | Mainnet (Archive) | `https://archive-rpc.lightsail.network/` |
| Ankr | Mainnet (Archive) | `https://rpc.ankr.com/stellar_soroban` |
