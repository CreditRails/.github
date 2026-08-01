# Pending DeFi Protocol Integrations

Research findings for adding more protocols to `indexer/src/config.ts` → `DEFI_CONTRACTS`
(currently only Blend and Soroswap are registered — see `scoring-data-sources.md`). Researched
2026-08-01 via Aquarius's own docs and general web search; verify again before hardcoding
anything, especially testnet addresses.

---

## Aqua (Aquarius) — real, addable

Aquarius is a real Soroban AMM. Source: [docs.aqua.network](https://docs.aqua.network/developers/code-examples/prerequisites-and-basics), [AquaToken/soroban-amm](https://github.com/AquaToken/soroban-amm).

| Network | AMM router/entrypoint contract |
|---|---|
| Mainnet | `CBQDHNBFBZYE4MKPWBSJOPIYLW4SFSXAXUTSXJN76GNKYVYPCKWC6QUK` |
| Testnet | `CBCFTQSPDBAIZ6R6PJQKSQWKNKWH2QIV3I4J72SHWBIK3ADRRAM5A6GD` (per Aquarius docs, updated Feb 2026) |

**Caveat:** Aquarius's own docs state the testnet address changes across testnet resets — an
earlier search turned up a different testnet address dated Dec 2024
(`CDGX6Q3ZZIDSX2N3SHBORWUIEG2ZZEBAAMYARAXTT7M5L6IXKNJMT3GB`), confirming it's not stable. The
mainnet address is consistent across sources and safe to add. **Re-verify the testnet address
directly from Aquarius's docs immediately before adding it** — don't copy the value above without
rechecking.

**To add:** register both as `{ protocol: "aqua", category: "amm", role: "router" }` in
`DEFI_CONTRACTS`, decode `invoke_host_function` calls the same way Soroswap is handled in
`indexer/src/defi.ts`.

---

## Sava — not a real protocol

No contract, docs, project site, or any reference found under "Sava" + Stellar/Soroban savings.
This name appears to be aspirational/placeholder branding carried over from early marketing copy
(`data/integrations.md`), not a real integration target. **Don't add — there's nothing to point
at.** Recommend removing "Sava" from `data/integrations.md` and any other reference docs that
still list it as "planned," or replacing it with a real savings-protocol candidate if one is
identified later.

---

## StellarX — not Soroban, different detection path entirely

StellarX is a trading UI over Stellar's **classic DEX (SDEX)** — an order-book system that
predates Soroban. Soroban contracts cannot interact with SDEX at all; it's a fully separate
system. There is no contract address to register in `DEFI_CONTRACTS` because SDEX activity isn't
`invoke_host_function` calls — it's classic `manage_buy_offer` / `manage_sell_offer` / `create_passive_sell_offer`
operations, which `indexer/src/horizon.ts` → `fetchOperations()` already pulls (visible in
`defiInteractionCount`-adjacent signals like `distinctAssets`), just not currently classified as
"DeFi participation" the way `invoke_host_function` calls are in `defi.ts`.

**To add "StellarX" style DEX activity:** this isn't a contract-registry addition — it needs new
logic in `defi.ts` (or a new signal entirely) that counts classic DEX offer operations from
`fetchOperations()`. Worth doing, but it's a different code path than Blend/Soroswap/Aqua, not
just another `DEFI_CONTRACTS` entry.
