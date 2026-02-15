# StableLink

## 1. Project Overview

StableLink is an Etherlink-native stablecoin payment infrastructure platform designed for Asia-based freelancers and small agencies.

It enables users to:
- Create programmable stablecoin invoices
- Accept payments in USDC
- Automatically split funds (platform fee + recipients)
- Withdraw funds non-custodially
- Manage organizations and team roles
- Integrate via REST APIs and webhooks

StableLink is NOT a wallet.
StableLink is NOT custodial.
StableLink is financial infrastructure built on Etherlink.

---

## 2. Vision & Alignment

## Core Alignment

- Built natively on Etherlink (EVM compatible)
- Focused on stablecoin payments (USDC primary)
- Asia-first freelancer & agency use case
- Non-custodial architecture
- Infrastructure-grade thinking

## Problem Being Solved

Cross-border payments for freelancers are:
- Slow (3–7 day settlement)
- Expensive (FX + remittance fees)
- Opaque
- Custodial

StableLink solves this by:
- Enabling direct on-chain stablecoin invoices
- Providing instant settlement
- Maintaining wallet custody
- Enabling programmable splits

---

## 3. High-Level Architecture

```
Frontend (Next.js)
        |
Backend (Node.js + Express)
        |
Database (PostgreSQL or SQLite)
        |
Smart Contracts (Solidity)
        |
Etherlink Blockchain
```

- Money lives on-chain.
- Metadata lives off-chain.
- Backend never touches user funds.

---

## 4. Smart Contract Design

### Contract Name

`InvoicePayments.sol`

### Token Support

- USDC primary
- ERC20-compatible token configurable

### Data Structures

```solidity
struct Split {
    address recipient;
    uint16 percentage; // basis points (10000 = 100%)
}

struct Invoice {
    address creator;
    address token;
    uint256 amount;
    bool paid;
    bool withdrawn;
}
```

**Storage**

- `mapping(uint256 => Invoice) public invoices;`
- `mapping(uint256 => Split[]) public invoiceSplits;`
- `mapping(address => mapping(address => uint256)) public balances;` — balances[user][token] => withdrawable amount

**Core Functions**

- **createInvoice** — Parameters: amount, token, splits[]. Validations: amount > 0, splits total == 10000. Emits: InvoiceCreated
- **payInvoice** — Parameters: invoiceId. Logic: TransferFrom payer → contract, mark invoice as paid, calculate split balances, update balances mapping. Emits: InvoicePaid
- **withdraw** — Parameters: token, amount. Logic: Ensure sufficient balance, transfer token to caller, update balance. Emits: FundsWithdrawn

**Design Principles**

- No upgradeability for MVP
- No custody logic
- Minimal complexity
- Clear event emission
- Etherlink deployment only

---

## 5. Backend Architecture (Node.js)

### Stack

- Node.js
- Express
- ethers.js
- PostgreSQL (preferred) or SQLite
- dotenv
- CORS
- JWT (future)

### Responsibilities

- Store invoice metadata
- Index blockchain events
- Power dashboard APIs
- Manage organization & team logic
- API key authentication
- Webhook dispatch

**Backend NEVER:** Holds funds, signs transactions, custodies tokens.

### Database Schema

**Users:** id, email, wallet_address, created_at

**Organizations:** id, name, primary_wallet, default_platform_fee, created_at

**OrganizationMembers:** id, organization_id, user_id, role (admin / finance / viewer), status

**Invoices:** id (internal), onchain_invoice_id, organization_id, creator_wallet, client_name, client_email, token, amount, status (draft / paid / withdrawn), tx_hash, created_at, paid_at

**APIKeys:** id, organization_id, live_key, test_key, created_at

**Webhooks:** id, organization_id, url, subscribed_events, created_at

---

## 6. REST API Design

### Authentication

- `Authorization: Bearer API_KEY`

### Endpoints

- POST /api/invoices
- GET /api/invoices
- GET /api/invoices/:id
- POST /api/webhooks
- GET /api/organization
- POST /api/team/invite

### Example Create Invoice Payload

```json
{
  "amount": 1000,
  "token": "USDC",
  "client_name": "Acme Inc",
  "splits": [
    { "wallet": "0xabc...", "percentage": 9700 },
    { "wallet": "0xdef...", "percentage": 300 }
  ]
}
```

---

## 7. Webhooks

**Events:**

- invoice.created
- invoice.paid
- withdrawal.completed

Backend listens to smart contract events via ethers.js. Dispatches POST requests to registered webhook URLs.

**Optional:** HMAC signature verification

---

## 8. Frontend Architecture (Next.js)

### Pages

- Landing
- Dashboard
- Create Invoice
- Public Invoice Payment
- Withdraw
- Invoices
- Settings & Organization
- API & Developer

### Wallet Integration

- MetaMask
- Network check (Etherlink)
- Auto-switch support
- Address short display

### UX Principles

- Stripe-like clarity
- Futuristic but professional
- No crypto jargon
- Strong transaction states
- Clear success/failure feedback

---

## 9. Role-Based Access

**Roles:**

- **Admin:** Full control, withdraw, manage members
- **Finance:** Create invoices, view reports, no team management
- **Viewer:** Read-only

**Enforced in:** Backend middleware, UI visibility logic

---

## 10. Security Principles

- Non-custodial smart contract
- No private key storage
- Role-based backend access
- API key rotation
- Webhook verification (future)
- No user passwords (wallet-first auth)

---

## 11. Revenue Model

**Platform Fee:**

- Default 3%
- Auto-applied in split
- Configurable per organization

Revenue is distributed via smart contract split logic.

---

## 12. What Is NOT Included in MVP

- Fiat on/off ramp
- KYC
- Token swaps
- Multi-chain deployment
- Smart contract upgradeability
- Audits
- Mobile app

---

## 13. Grant Eligibility Positioning

StableLink demonstrates:

- Real MVP functionality
- Etherlink-native deployment
- Financial infrastructure thinking
- Role-based SaaS maturity
- API programmability
- Clear monetization model

This is not a hackathon demo. This is early-stage financial infrastructure.

---

## 14. Development Order

1. Smart Contract (deploy to Etherlink testnet)
2. Backend event indexing
3. Invoice creation flow
4. Payment flow
5. Withdrawal logic
6. Organization & roles
7. API system
8. Webhooks
9. Final polish & demo recording

---

## 15. Demo Flow for Grant Review

1. Connect wallet
2. Create invoice
3. Open payment link
4. Pay invoice
5. See split distribution
6. Withdraw funds
7. Show API page

Duration: 3 minutes max.

---

## 16. Core Philosophy

StableLink is:

- Non-custodial
- Infrastructure-focused
- Asia-first
- Etherlink-native
- Scalable beyond MVP
- Built for long-term startup growth
