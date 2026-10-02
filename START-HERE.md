# Nexora v2 — React + Node.js + PostgreSQL

This is a NEW standalone version of your project. Open THIS folder in VS Code. Do not overlay it onto the older Cloudflare/Vinext project.

## What changed

- One Log in button; duplicate Get started button removed.
- Demo account IDs/passwords removed from the browser UI and frontend source.
- Customer login opens verification first. Admin login opens client management.
- Only the administrator can approve/reject documents.
- Approval is checked again by the Node backend before a purchase.
- Each account has independent numbers, wallet balance, documents and payments.
- Number purchases deduct the wallet balance only after a correct six-digit PIN.
- Five incorrect PIN attempts lock PIN operations for 15 minutes.
- Admin can see login activity, payments and orders, create clients and block/unblock access.
- React/Vite frontend can be hosted on Netlify. Node backend and PostgreSQL are separate.
- No ChatGPT sign-in or Cloudflare D1/R2 dependency remains.

## 1. Install

Requirements: Node.js 22.13+ and PostgreSQL (your existing PostgreSQL 18 is suitable).

Extract this ZIP. In VS Code open the `nexora-fullstack` folder containing the ROOT `package.json`.

In the VS Code PowerShell terminal, run:

```powershell
npm.cmd install
```

Copy the backend configuration:

```powershell
Copy-Item server\.env.example server\.env
```

Open `server/.env`. Change this line to YOUR PostgreSQL password:

```dotenv
DB_PASSWORD=YOUR_POSTGRES_PASSWORD
```

Keep DB_HOST=localhost, DB_PORT=5432, DB_NAME=nexora and DB_USER=postgres unless your PostgreSQL setup differs.
Set your own ADMIN_EMAIL and ADMIN_PASSWORD. Minimum admin password length: 12 characters.
The .env file is private and must not be uploaded to GitHub or Netlify.

## 2. Create database and accounts

Make sure your PostgreSQL Windows service is running. Then run:

```powershell
npm.cmd run db:setup
```

With the local postgres administrator user this creates the `nexora` database if it does not already exist, adds tables and seeds five sample customer accounts plus the administrator.
Re-running setup preserves existing balances/passwords. Editing .env after seeding does not reset existing account passwords or PINs.
Demo credentials are printed in the terminal; they are not shown on the login page.
This is a separate PostgreSQL database. Records in the old D1/local Cloudflare project are not automatically migrated.

## 3. Run both frontend and backend

From the ROOT project folder:

```powershell
npm.cmd run dev
```

Open:

```text
http://localhost:5173
```

The backend runs on port 5000. If port 5000 is already used, stop the other project first. Keep the terminal open.
No `/signin-with-chatgpt` step is needed. The Node backend performs account login directly.

Alternative: use two terminals:

```powershell
npm.cmd run dev -w server
```

```powershell
npm.cmd run dev -w client
```

## 4. Test the complete flow

The backend terminal prints the five demonstration emails and their password/PIN when PRINT_DEMO_CREDENTIALS=true and NODE_ENV is not production.
Default sample password: Nexora@123. Default sample wallet PIN: 123456.
Default admin login is configured in server/.env; the example is admin@nexora.test / ChangeAdmin@12345. Change it BEFORE db:setup.

1. Log in as a customer. Verification opens first.
2. Upload a SAMPLE passport, driving licence or identity card (PDF/JPG/PNG, maximum 5 MB).
3. Log out. Log in using the separate administrator account.
4. In Admin · Clients, download/review the sample document and click Approve or Reject.
5. Log back in as that customer; approved status is saved. Choose a country and service.
6. Checkout shows that customer's wallet balance. Enter that customer's six-digit wallet PIN.
7. The server checks approval, PIN and funds, deducts the country price, and creates an order for THAT account.
8. Check My numbers, Wallet & payments and Order history. Simulate an SMS if needed.
9. Log in as another customer: their balance, documents and numbers remain independent.

In Wallet & payments, Change wallet PIN requires the current PIN.
Admin may create new customer accounts with separate temporary passwords and PINs. New wallets start at ₹0 and verification is unverified.

## 5. Payment modes

Default `PAYMENT_MODE=demo`: top-ups are simulated and no money is collected. Every top-up is saved, and duplicate requests do not credit a wallet twice.
Number purchases use the wallet plus PIN. Purchased numbers and SMS remain demonstration data, not real telecom inventory.

Optional `PAYMENT_MODE=razorpay-test`: put your OWN Razorpay test keys into server/.env:

```dotenv
PAYMENT_MODE=razorpay-test
RAZORPAY_KEY_ID=rzp_test_YOUR_KEY
RAZORPAY_KEY_SECRET=YOUR_SECRET
RAZORPAY_WEBHOOK_SECRET=YOUR_WEBHOOK_SECRET
```

The frontend opens Razorpay Standard Checkout for wallet top-ups. The server creates the order, verifies the returned HMAC signature, fetches the payment and credits the wallet only after a captured INR payment with the expected amount. Configure automatic capture in Razorpay test settings.
For recovery when the checkout tab closes, configure the `payment.captured` webhook at `https://YOUR_BACKEND/api/payments/webhook`, using the same webhook secret.
LIVE keys are intentionally not enabled: real number delivery is not connected. Razorpay external calls require your test keys and were not tested against your account.
Do not put API secrets in client code or Netlify client environment variables.

## 6. Netlify frontend + separate backend + PostgreSQL

See NETLIFY-DEPLOY.md for exact settings.
Netlify runs the React frontend; it does not run the long-lived Express server in this package.
Host Node separately and use your separately hosted PostgreSQL database.

## Main files

- client/src/App.tsx — React landing page and dashboards
- client/src/styles.css — green theme and animations
- client/vite.config.ts — local API proxy to port 5000
- server/src/app.js — Node/Express API, permissions, uploads, wallet/PIN and payment routes
- server/src/security.js — password/PIN hashing and session helpers
- server/src/db.js — PostgreSQL pool and transactions
- server/sql/001-schema.sql — database schema
- server/scripts/setup-db.js — database setup and demo seeding
- server/.env.example — backend configuration template
- netlify.toml — Netlify build configuration
- client/scripts/netlify-redirects.mjs — production API proxy rules

## Troubleshooting

ENOENT package.json: terminal is outside nexora-fullstack. Open the correct folder.
ECONNREFUSED 5432: start PostgreSQL service.
28P01: correct DB_USER / DB_PASSWORD in server/.env.
EADDRINUSE 5000: stop the other backend occupying port 5000.
401 Please log in: login on this website normally; do not use the old platform sign-in link.
Database unavailable / relation users missing: run npm.cmd run db:setup before dev.
Netlify API returns HTML: NETLIFY_BACKEND_URL was not configured or the proxy rules were not built; redeploy after configuring the backend URL.

## Tests

Tests deliberately require a SEPARATE database whose name ends with `_test`. They truncate that test database only. Do not point tests at nexora or your real data.

```powershell
$env:TEST_DATABASE_URL="postgresql://postgres:YOUR_PASSWORD@localhost:5432/nexora_test"
npm.cmd test
```

Create nexora_test in PostgreSQL before using these commands. API integration tests cover account isolation, admin permissions, uploads/reviews, PIN checks, overspending prevention, duplicate payment/refund protection, blocking and logout.
