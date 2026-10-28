# MozaikPay Commerce Engine: Full-Stack Integration & Merchant POS Suite

A production-grade, enterprise-ready payment gateway integration showcasing the full lifecycle of **MozaikPay** on the Solana blockchain. This project demonstrates how modern Web3 merchants, SaaS platforms, and Point-of-Sale (POS) operators can leverage MozaikPay to accept any SPL token (USDC, SOL, USDT, BONK) while settling instantaneously into their treasury token of choice with automated order reconciliation, webhook idempotency, and interactive checkout widgets.

---

## 1. Executive Summary & Overview

Traditional crypto payment processors suffer from checkout friction, multi-token fragmentation, and volatile asset handling. **MozaikPay** solves this on Solana by providing instant settlement, low fees, unified routing, and seamless merchant abstraction.

This repository demonstrates an end-to-end implementation of MozaikPay:
- **E-Commerce Checkout Widget**: Client-side drop-in component supporting wallet-connect, Solana Pay QR codes, and any-token routing.
- **MozaikPay Merchant SDK**: Robust TypeScript client implementing dynamic invoice creation, real-time transaction verification, and slippage-protected swap settlements.
- **Backend Order Management & Webhook Engine**: Express/Node.js microservice validating cryptographically signed webhooks (`X-Mozaik-Signature`), handling idempotency keys, and automating inventory fulfillment.
- **POS / Merchant Terminal UI**: Built-in responsive dashboard for generating dynamic payment links, tracking settlement streams, and inspecting webhook event logs.

---

## 2. Architecture & Transaction Lifecycle

```
       +------------------+                    +---------------------+
       |   Customer App   |                    |  Merchant Backend   |
       +--------+---------+                    +----------+----------+
                |                                         |
         1. Initiate Checkout                             |
                |                                         |
                v                                         |
       +------------------+    2. Create Invoice / Link   |
       |  MozaikPay Modal | <-----------------------------+
       +--------+---------+                               |
                |                                         |
         3. Sign & Broadcast TX                           |
                |                                         |
                v                                         v
       +------------------+                   +----------------------+
       |  Solana Cluster  |                   | Mozaik Webhook Relay |
       |  (RPC / Anchor)  |                   +-----------+----------+
       +--------+---------+                               |
                |                                         |
         4. On-chain Finality                             |
                +-----------------------------------------+
                |
                v
       +------------------+    5. Verified Webhook Payload
       | Merchant Webhook | <------------------------------
       |     Receiver     |
       +--------+---------+
                |
         6. Idempotent State Transition & Order Dispatch
```

### Flow Breakdown
1. **Invoice Initialization**: Merchant backend creates a MozaikPay transaction session with merchant metadata, accepted tokens, and target settlement address.
2. **Dynamic Routing**: MozaikPay constructs an optimized transaction instruction (utilizing direct SPL transfers or DEX routing via Jupiter/Raydium if payment token differs from settlement token).
3. **Execution & Commitment**: The customer signs via Phantom, Solflare, or Solana Mobile Wallet Adapter.
4. **On-Chain Settlement**: The transaction finalizes within 400ms on Solana with sub-cent transaction fees.
5. **Webhook Dispatch & Cryptographic Verification**: MozaikPay dispatches an HMAC-SHA256 signed event payload to the merchant webhook endpoint.
6. **Order Fulfillment**: Merchant verifies signature, enforces idempotency, updates the database, and issues goods/receipts.

---

## 3. Key Features

- **Multi-Token Acceptance**: Accept USDC, SOL, USDT, BONK, and custom SPL tokens seamlessly.
- **Auto-Settlement**: Real-time routing directly into merchant USDC or SOL accounts.
- **Solana Pay & Mobile Compatibility**: Generates standard Solana Pay transaction URLs and responsive QR codes.
- **Cryptographic Webhook Security**: Secure HMAC-SHA256 signature verification preventing replay and tampering attacks.
- **Idempotency Guard**: Built-in cache/store tracking `invoice_id` and `tx_signature` to prevent duplicate fulfillment.
- **Production-Ready POS Dashboard**: Built-in web console for cashier operations, manual link generation, and audit logging.

---

## 4. Project Structure

```
├── src/
│   ├── sdk/
│   │   ├── MozaikClient.ts       # Core MozaikPay protocol client wrapper
│   │   └── types.ts              # Strict TypeScript definitions & interfaces
│   ├── server/
│   │   ├── index.ts              # Express API server & static host
│   │   ├── routes/
│   │   │   ├── checkout.ts       # Invoice generation and status polling
│   │   │   └── webhook.ts        # Cryptographic webhook receiver & verification
│   │   └── storage/
│   │       └── db.ts             # In-memory transactional order repository
│   └── public/
│       ├── index.html            # Merchant POS & E-commerce checkout app
│       ├── app.js                # Frontend checkout interaction & QR rendering
│       └── style.css             # Tailwind/Modern UI styles
├── package.json                  # Dependencies & scripts
└── tsconfig.json                 # TypeScript compiler configuration
```

---

## 5. Getting Started & Installation

### Prerequisites
- Node.js >= 18.0.0
- npm or yarn or pnpm
- Solana Wallet (e.g., Phantom or Solflare) connected to Devnet/Mainnet

### Installation

1. Clone or extract the repository:
   ```bash
   git clone https://github.com/your-org/mozaikpay-showcase.git
   cd mozaikpay-showcase
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Configure environment variables:
   ```bash
   cp .env.example .env
   ```
   Edit `.env`:
   ```env
   PORT=3000
   SOLANA_NETWORK=devnet
   SOLANA_RPC_URL=https://api.devnet.solana.com
   MOZAIK_API_KEY=mzk_test_9f82c1a84b39201e74
   MOZAIK_WEBHOOK_SECRET=whsec_8849b201cd9a21e48bc702319fba
   MERCHANT_WALLET_ADDRESS=8YwK8nS4V1s2Yx9Q8YjW6uF1g9vQ8YwK8nS4V1s2Yx9Q
   SETTLEMENT_CURRENCY=USDC
   ```

4. Build and start the service:
   ```bash
   # Development mode with live reload
   npm run dev

   # Production build and run
   npm run build
   npm start
   ```

5. Open your browser:
   Visit `http://localhost:3000` to interact with the Merchant POS & Checkout Hub.

---

## 6. API Reference

### Create Invoice
- **Endpoint**: `POST /api/checkout/create-invoice`
- **Request Body**:
  ```json
  {
    "amount": 25.50,
    "currency": "USDC",
    "customer": {
      "email": "alex@solana.org",
      "name": "Alex Miller"
    },
    "metadata": {
      "orderId": "ORD-94821",
      "sku": "SOL-HOODIE-L"
    }
  }
  ```
- **Response**:
  ```json
  {
    "success": true,
    "invoice": {
      "id": "inv_a83f98c1",
      "paymentUrl": "https://pay.mozaikpay.com/pay/inv_a83f98c1",
      "solanaPayUrl": "solana:https://api.mozaikpay.com/v1/tx/inv_a83f98c1",
      "amount": 25.50,
      "currency": "USDC",
      "expiresAt": "2025-05-18T14:30:00.000Z"
    }
  }
  ```

### Webhook Ingestion
- **Endpoint**: `POST /api/webhook/mozaik`
- **Headers**:
  - `X-Mozaik-Signature`: HMAC SHA-256 hash of the payload using `MOZAIK_WEBHOOK_SECRET`
  - `X-Mozaik-Event`: `invoice.paid` | `invoice.expired` | `invoice.failed`
- **Response**: `200 OK` with `{ "received": true }`

---

## 7. Security & Best Practices

1. **HMAC Signature Verification**: Prevents spoofing by verifying incoming webhook headers against the merchant's private shared secret before taking stateful actions.
2. **Idempotency Guard**: Prevents double-spending or duplicate order fulfillment across duplicate webhook retry attempts.
3. **Transaction Signature Verification**: Performs independent on-chain confirmation checks on Solana RPC using commitment level `confirmed` or `finalized`.
4. **Zero Key Custody**: The merchant never touches private keys. Payment flows go directly through non-custodial smart instructions to the designated recipient treasury account.