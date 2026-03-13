# Veil

A passkey-powered smart wallet on the Stellar Soroban blockchain. Users authenticate with their device biometrics (Face ID, fingerprint, Windows Hello) instead of seed phrases or private keys.

## How it works

Veil combines WebAuthn (the browser passkey standard) with a Soroban custom account contract. When a user registers, a P-256 keypair is created on their device and the public key is stored in the wallet contract. To authorize a transaction, the user's device signs the Soroban authorization payload with their passkey. The contract verifies the full WebAuthn assertion on-chain — including the challenge binding and the ECDSA signature — before approving any action.

```
User device                        Stellar network
──────────────────────────────     ─────────────────────────────
1. register()
   └─ WebAuthn credential create
   └─ extract P-256 public key ──► deploy wallet contract
                                        └─ store public key

2. signAuthEntry(payload)
   └─ WebAuthn assertion
   └─ DER → raw sig conversion
   └─ return {pubkey, authData,
      clientDataJSON, sig}    ────► __check_auth()
                                        └─ verify challenge in clientDataJSON == payload
                                        └─ compute SHA256(authData || SHA256(clientDataJSON))
                                        └─ verify P-256 ECDSA signature
                                        └─ approve / reject
```

## Project structure

```
veil/
├── contracts/
│   └── invisible_wallet/          # Soroban smart contract (Rust)
│       ├── src/
│       │   ├── lib.rs             # Contract entry points + __check_auth
│       │   ├── auth.rs            # WebAuthn ES256 verification logic
│       │   └── storage.rs         # Signer and guardian storage
│       └── Cargo.toml
└── sdk/
    ├── src/
    │   ├── useInvisibleWallet.ts  # React hook — register, login, signAuthEntry
    │   ├── utils.ts               # Crypto utilities (DER→raw, pubkey extraction, SHA256)
    │   └── index.ts               # Package exports
    ├── package.json
    └── tsconfig.json
```

## Tech stack

| Layer | Technology |
|---|---|
| Smart contract | Rust, Soroban SDK v20, p256 crate (ECDSA), sha2 |
| Authentication | WebAuthn / FIDO2 (ES256 / P-256) |
| Client SDK | TypeScript, React hooks, Web Crypto API |
| Blockchain | Stellar (Soroban smart contracts) |

## Getting started

### Prerequisites

- [Rust](https://rustup.rs/) with the `wasm32-unknown-unknown` target
- [Stellar CLI](https://developers.stellar.org/docs/tools/stellar-cli)
- Node.js 18+

### Build the contract

```bash
cd contracts/invisible_wallet
cargo build --target wasm32-unknown-unknown --release
```

### Run contract tests

```bash
cd contracts/invisible_wallet
cargo test
```

### Build the SDK

```bash
cd sdk
npm install
npm run build
```

## Usage

```tsx
import { useInvisibleWallet } from 'invisible-wallet-sdk';

function App() {
  const wallet = useInvisibleWallet(FACTORY_CONTRACT_ADDRESS);

  // Register a new passkey wallet
  await wallet.register('alice');

  // Sign a Soroban authorization entry
  const sig = await wallet.signAuthEntry(signaturePayload); // Uint8Array (32 bytes)
  // sig = { publicKey, authData, clientDataJSON, signature }
  // Encode sig as Vec<Val> and attach to the Soroban auth entry
}
```

## Signature format

The contract's `__check_auth` expects the signature field to be a `Vec<Val>` with four elements:

| Index | Type | Description |
|---|---|---|
| 0 | `BytesN<65>` | Uncompressed P-256 public key (`0x04 \|\| x \|\| y`) |
| 1 | `Bytes` | WebAuthn `authenticatorData` |
| 2 | `Bytes` | WebAuthn `clientDataJSON` (must contain `base64url(signature_payload)` as challenge) |
| 3 | `BytesN<64>` | Raw P-256 ECDSA signature (`r \|\| s`) |

## Roadmap

- [x] Phase 1 — Contract compiles, error types, ECDSA verification, unit tests
- [x] Phase 2 — Full WebAuthn pipeline (DER→raw, real pubkey extraction, challenge binding)
- [ ] Phase 3 — Factory contract + deterministic wallet deployment
- [ ] Phase 4 — Testnet integration, end-to-end demo
- [ ] Phase 5 — Guardian recovery, multi-signer, replay protection

## License

MIT
