# stellar-passkey-auth

> Passkey and social login authentication for Stellar dApps — let users sign transactions
> with Face ID, Touch ID, or Google without ever seeing a seed phrase.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)
[![Drips Wave Program](https://img.shields.io/badge/Drips-Wave%20Program-8B5CF6)](https://drips.network)

---

## Overview

The single biggest onboarding barrier in Web3 is key management. `stellar-passkey-auth`
removes it entirely — users register with a passkey (biometric or hardware key) or their
Google/GitHub account, and the module handles the mapping to a Soroban smart wallet
contract that accepts their credential as a valid transaction signer.

No seed phrases. No browser extensions. Just tap and sign.

---

## Technical Architecture

- **WebAuthn / Passkeys:** `@simplewebauthn/server` handles registration and
  authentication ceremonies server-side; `@simplewebauthn/browser` drives the
  browser-side credential creation and assertion flows
- **Smart Wallet Contract:** Rust/Soroban contract in `contracts/src/` implementing
  the Stellar smart wallet interface — accepts secp256r1 signatures from passkey
  credentials as valid transaction authorization
- **Social Login:** OAuth 2.0 via `passport.js` (Google, GitHub providers) — maps
  OAuth identity to a deterministic encrypted keypair, enabling familiar login UX
  without sacrificing self-custody
- **Session Management:** JWT-based session layer with refresh token rotation,
  storing the derived wallet address and credential ID (never the private key)
- **Framework Adapters:** Drop-in middleware for Next.js API routes and Express.js
  backends; React hooks for browser-side passkey ceremonies

---

## 💧 Drips Wave Program

This repository is an active participant in the
**[Drips Wave Program](https://drips.network)** — a funding mechanism that rewards
open-source contributors for resolving scoped GitHub issues with on-chain streaming
payments.

### How to Contribute & Earn

**Step 1 — Register on Drips**
Visit [drips.network](https://drips.network) and connect your Ethereum-compatible wallet.
Your wallet address is where reward streams will be sent.

**Step 2 — Browse Open Issues**
Head to the [Issues tab](../../issues). Issues are labeled by complexity tier:

| Label           | Complexity | Typical Scope                                                              |
|-----------------|------------|----------------------------------------------------------------------------|
| `drips:trivial` | Trivial    | Improve JSDoc, add usage example, write a unit test for a utility function |
| `drips:medium`  | Medium     | New OAuth provider, session refresh logic, credential storage improvements |
| `drips:high`    | High       | Hardware key support, multisig passkey setup, mobile React Native adapter  |

**Step 3 — Claim an Issue**
Comment `/claim` on your chosen issue. The maintainer will assign it.
Issues are first-come, first-served.

**Step 4 — Submit a PR**
Open a Pull Request with `Closes #XX` in the description. Include unit tests for all
new logic and update relevant docs and examples.

**Step 5 — Earn Rewards**
Your wallet begins receiving a continuous Drips stream upon PR merge.
No invoices, no delays.

---

## Project Structure
```
stellar-passkey-auth/
├── src/
│   ├── passkey/              # WebAuthn registration + authentication ceremonies
│   ├── social/
│   │   ├── providers/        # Google, GitHub OAuth strategy configs
│   │   └── callbacks/        # OAuth callback handlers, identity → wallet mapping
│   ├── wallet/               # Smart wallet contract client, address derivation
│   ├── session/              # JWT issue, verify, refresh token rotation
│   ├── api/
│   │   ├── routes/           # Auth routes (register, login, verify, logout)
│   │   └── middleware/       # Session guard, CSRF protection
│   ├── utils/                # Crypto helpers, base64url, credential serializers
│   └── errors/               # Typed auth error classes
├── contracts/
│   ├── src/                  # Rust/Soroban smart wallet contract (secp256r1 auth)
│   └── tests/                # Rust unit tests
├── examples/
│   ├── nextjs/               # Full Next.js App Router integration example
│   └── express/              # Express.js backend integration example
├── tests/
│   ├── unit/                 # Passkey ceremony logic, JWT utils, error handling
│   └── integration/          # Full auth flow tests against local Stellar node
├── docs/                     # Setup guides, architecture diagrams
├── config/                   # Zod env schemas
├── package.json
├── tsconfig.json
└── README.md
```

---

## Quick Start
```bash
npm install stellar-passkey-auth
```

### Next.js Example
```typescript
// app/api/auth/register/route.ts
import { registerPasskey } from 'stellar-passkey-auth/nextjs';

export async function POST(req: Request) {
  const { userId, username } = await req.json();
  const { options, walletAddress } = await registerPasskey({ userId, username });
  return Response.json({ options, walletAddress });
}
```

### Sign a Stellar Transaction
```typescript
import { signWithPasskey } from 'stellar-passkey-auth/browser';

const signedXdr = await signWithPasskey({
  transactionXdr: unsignedTx.toXDR(),
  walletAddress: 'CXXX...',
});
```

---

## License

MIT © stellar-passkey-auth contributors
