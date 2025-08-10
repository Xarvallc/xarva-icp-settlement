# Xarva – ICP Telecom Settlement Canisters

**Xarva** brings real-time, on-chain B2B invoice settlement to the Internet Computer (ICP).  
Our goal: replace legacy 2–5 day telecom reconciliations with **sub-5-minute**, auditable flows.

> Grant track: **Apps & Open Internet Services** — $25k (≤4 months)

---

## Why ICP
- **Low-latency canister execution** for near-real-time workflows
- **Cost-efficient, scalable storage** for invoice/settlement logs
- **HTTPS outcalls & chain-key crypto** for safe price/KYC checks without 3rd-party servers

---

## High-Level Architecture

```
+----------------+          +---------------------+
| Carrier A DApp |<--HTTP-->|  Settlement Portal  |  (minimal React demo)
+----------------+          +----------+----------+
                                      |
                                      v
                           +----------+-----------+
                           |  Settlement Canister |
                           |  - Invoice registry  |
                           |  - State machine     |
                           |  - Fees/FX adapter   |
                           +----------+-----------+
                                      |
                           +----------+-----------+
                           |  ICRC-1 Token/Ledger |  (demo settlement asset)
                           +----------------------+
```

### Canister roles (initial release)
- **Settlement**: creates invoices, approves, settles, records events.
- **ICRC-1 adapter**: pays/escrows settlement amounts in a demo token (can swap for XRV later).
- **FX/KYC stubs**: HTTPS outcalls to public feeds (price) and mock KYC provider (for demo).

---

## Repository Layout

```
/canisters/settlement/        # Rust canister (ic-cdk)
/docs/                        # Grant milestones, API docs, diagrams
/scripts/                     # Demo scripts
README.md
dfx.json
Cargo.toml                    # workspace
```

> Language: Rust (ic-cdk). We’ll add Motoko examples where helpful.

---

## Quick Start (local)

**Prereqs**
- `dfx` ≥ 0.20, Rust stable, Node 18+
- `wasm32-unknown-unknown` target: `rustup target add wasm32-unknown-unknown`

**Run**
```bash
# 1) clone & start local replica
git clone https://github.com/YOUR-ORG/xarva-icp-settlement.git
cd xarva-icp-settlement
dfx start --clean --background

# 2) deploy canisters
dfx deploy settlement
```

**Sample calls**
```bash
# Create an invoice of 123.45 units between carriers
dfx canister call settlement create_invoice   '("INV-001", "carrierA", "carrierB", 123_450_000:nat, 1:nat)'

# Approve by payer
dfx canister call settlement approve_invoice '("INV-001", "carrierB")'

# Settle (moves tokens in later milestones; stubbed here)
dfx canister call settlement settle_invoice '("INV-001")'

# Read back
dfx canister call settlement get_invoice '("INV-001")'
dfx canister call settlement list_invoices '(0:nat, 50:nat)'
```

---

## Core API (Candid outline)

```did
type InvoiceId = text;
type CarrierId = text;

type Invoice = record {
  id: InvoiceId;
  from: CarrierId;
  to: CarrierId;
  amount : nat;       // 6 decimals
  asset  : nat;       // asset id (1 = demo ICRC-1 token)
  status : variant { Created; Approved; Settled; Cancelled };
  created_at : nat64;
  updated_at : opt nat64;
};

service : {
  create_invoice : (InvoiceId, CarrierId, CarrierId, nat, nat) -> (Invoice);
  approve_invoice : (InvoiceId, CarrierId) -> (Invoice);
  settle_invoice : (InvoiceId) -> (Invoice);
  get_invoice : (InvoiceId) -> (opt Invoice);
  list_invoices : (nat, nat) -> (vec Invoice); // paging
}
```

> Final signatures may change slightly during M1 hardening.

---

## Security & Upgrades
- Stable structures (to be added in M2) for upgrade safety.
- Role-based access (carrier principals).
- Event log with immutable receipts per action.
- Benchmark targets: median settle < 5s locally; document mainnet timings.

---

## Roadmap (Grant Milestones)
See [`/docs/grant_milestones.md`](./docs/grant_milestones.md) for the full, testable scope, timelines, and acceptance criteria.

---

## Contributing
Issues / PRs welcome. Please open an issue before large changes.

## License
Apache-2.0 (see `LICENSE`).
