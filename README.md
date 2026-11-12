# PayShield AI

**Detect Threats. Protect Payments. Build Trust.**

A payment security platform. It checks each payment before money moves: duplicate detection, an explainable fraud score, a protection gate (proceed / confirm / hold / block), PayPal order processing with verified webhooks, a live dashboard, and an AI investigation assistant.

> Rules decide, AI explains. Scores and decisions come from a deterministic engine. The LLM only rewrites explanations and answers questions over stored records.

---

## 🚀 Quick Start (5 minutes)

### Step 0: What you need

| Tool | Check it's installed | Install (macOS) |
|---|---|---|
| Node.js 20+ | `node -v` | `brew install node` or [nodejs.org](https://nodejs.org) |
| PostgreSQL 14+ | `psql --version` | `brew install postgresql@16 && brew services start postgresql@16` |

No Postgres? Use Docker instead: see [Option B](#option-b-using-docker-for-postgres).

### Step 1: Open the project folder

```bash
cd payshield-ai
```

Full path on the original machine: `/Users/kartikshakya/paytm-docs/payshield-ai`

### Step 2: Create the database

```bash
createdb payshield
```

### Step 3: Create the server config file

```bash
cp server/.env.example server/.env
```

Open `server/.env` in any editor and change two lines:

```env
DATABASE_URL=postgresql://YOUR_MAC_USERNAME@localhost:5432/payshield?schema=public
JWT_SECRET=any-random-string-at-least-32-characters-long-here
```

- Replace `YOUR_MAC_USERNAME` with your username (run `whoami` to find it).
- For `JWT_SECRET`, you can generate one with: `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`

Leave everything else as it is. PayPal stays in **Simulated mode** by default, so no PayPal account is needed.

### Step 4: Start the server (Terminal 1)

```bash
cd server
npm install
npm run db:migrate
npm run db:seed
npm run dev
```

Wait until you see:

```
PayShield AI API listening on http://localhost:4000
```

Leave this terminal running.

### Step 5: Start the dashboard (Terminal 2, new window)

```bash
cd payshield-ai/client
npm install
npm run dev
```

Wait until you see `Local: http://localhost:5173`.

### Step 6: Open the app

👉 **http://localhost:5173**

Sign in with:

- **Email:** `demo@payshield.local`
- **Password:** `DemoPayShield1`

You now have 26 sample transactions, alerts, and charts to explore. Click **Demo mode** (top right) to run scenarios.

---

## ✅ Quick checks (optional)

Open these in your browser:

| URL | Should show |
|---|---|
| http://localhost:4000/api/health | `{"status":"ok", ...}` |
| http://localhost:5173 | Login page, then the dashboard |

---

## 🧭 What you can do in the app

| Page | What it's for |
|---|---|
| **Overview** | KPIs, 14-day risk chart, live feed |
| **Transactions** | Search and filter payments; click one for details and actions |
| **Fraud Detection** | Risk score gauge, indicators, thresholds, re-score a payment |
| **Investigations** | Suspicious reports, cases, and the AI assistant |
| **Security Alerts** | Acknowledge or resolve alerts |
| **Settings** | Risk thresholds, protection policy, notifications (admin can edit) |

**Try a protected payment:** click **New protected payment** → enter an amount, recipient, and reference → **Run security check** → follow the result.

**Try demo scenarios:** **Demo mode** → pick Normal, Suspicious, Duplicate, Failed, or Successful. Every demo record is labelled **SIMULATED**.

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---|---|
| `createdb: command not found` | Install Postgres (Step 0). Or run `brew services start postgresql@16` |
| `database "payshield" does not exist` | Run Step 2 (`createdb payshield`) |
| `Invalid environment configuration: JWT_SECRET ...` | Make `JWT_SECRET` at least 32 characters in `server/.env` |
| `P1001 Can't reach database server` | Postgres is not running. Start it (`brew services start postgresql@16`) |
| `password authentication failed` | Your `DATABASE_URL` user/password is wrong. Fix it in `server/.env` |
| `run db:seed` says "Sample data already exists" | That's fine, the data is already there. Skip it |
| Login page says "Invalid email or password" | Run `npm run db:seed` again in `server/` (it creates the demo user) |
| Port 4000 or 5173 already in use | Stop the other app, or change `PORT` in `server/.env` |
| Dashboard is blank or shows network errors | Make sure Terminal 1 (server) is still running |

---

## Option B: Using Docker for Postgres

If you prefer Docker and don't want Postgres installed:

```bash
docker run -d --name payshield-pg -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:16
docker exec payshield-pg createdb -U postgres payshield
```

Then in `server/.env` use:

```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/payshield?schema=public
```

---

## Daily use

Next time, you only need:

```bash
# Terminal 1
cd payshield-ai/server && npm run dev

# Terminal 2
cd payshield-ai/client && npm run dev
```

Stop either with `Ctrl + C`.

---

## Running the tests

```bash
cd server
npm test
```

27 tests: fraud-engine unit tests plus an end-to-end API suite on your local database. The tests create and delete their own users and leave the demo data alone.

---

## Connecting real PayPal Sandbox (optional)

By default the app runs in **Simulated mode** and makes no PayPal calls. To connect the PayPal Sandbox:

1. Log in at [developer.paypal.com](https://developer.paypal.com) → **Apps & Credentials** → turn on **Sandbox**.
2. Create an app and copy its **Client ID** and **Secret**.
3. In `server/.env`, set `PAYPAL_CLIENT_ID` and `PAYPAL_CLIENT_SECRET`. Keep `PAYPAL_BASE_URL=https://api-m.sandbox.paypal.com`.
4. For webhooks, create a webhook in the Sandbox dashboard. PayPal must reach your server, so use a tunnel such as ngrok for `localhost`. Subscribe to `CHECKOUT.ORDER.APPROVED`, `PAYMENT.CAPTURE.COMPLETED`, and `PAYMENT.CAPTURE.DENIED`. Copy its ID into `PAYPAL_WEBHOOK_ID`.
5. Restart the server. The startup banner should say `PayPal mode: SANDBOX`.

The server refuses to start if credentials are set but the URL is not a sandbox host. No real money moves.

The AI assistant works without a key (rule-based answers). To use an OpenAI-compatible model, set `OPENAI_API_KEY` in `server/.env`.

---

## Project layout

```
payshield-ai/
├── server/                  Backend: Express + TypeScript + Prisma (port 4000)
│   ├── .env.example         Copy to .env and edit
│   ├── prisma/schema.prisma Database tables
│   ├── src/
│   │   ├── routes/          API endpoints
│   │   ├── services/        Business logic (fraud engine, PayPal, AI, retries...)
│   │   └── middleware/      Auth, validation, rate limits
│   └── test/                Automated tests
└── client/                  Frontend: React + Vite + Tailwind (port 5173)
    └── src/
        ├── pages/           Dashboard pages
        └── components/      Shared UI, payment modal, drawers
```

### Payment flow

```
Analyze ─► Gate
            ├─ LOW ........ Proceed to PayPal order
            ├─ MEDIUM ..... Payer must confirm the warning
            ├─ HIGH ....... Held for manual review (admin)
            └─ CRITICAL ... Held internally, pending authorized review (admin)
```

A "block" is an internal PayShield hold. It is never shown as a PayPal block.

### Risk scoring

Ten rules add points up to 100. Each rule has a maximum weight, and all weights and thresholds are editable in **Settings**.

| Level | Score |
|---|---|
| Low | below 30 |
| Medium | 30 to 59 |
| High | 60 to 79 |
| Critical | 80 and above |

Rules include: unusual amount for this payer, many payments in a short time, repeated failed attempts, duplicate payment, new recipient with a high amount, previously flagged recipient, unknown device, unusual location, and off-hours activity.

---

## Security notes

- Secrets (`JWT_SECRET`, PayPal secret, AI key) stay in `server/.env`. They are never sent to the browser.
- No card numbers, CVVs, or PayPal tokens are stored.
- Passwords are hashed with bcrypt. Requests are rate-limited and validated.
- Analysts see only their own transactions. Admins can review held payments and change settings.
- The first account you register becomes admin.
- The demo login (`demo@payshield.local`) is for local development only. Change or delete it before sharing this with anyone else.

---

## Limitations

- Real PayPal Sandbox payments have not been run yet; they need your Sandbox credentials (see above).
- Redis is not used: rate limits are in memory and retries run inside the server process. Fine for a single instance.
- Email notifications are not implemented. In-app alerts and notifications work.
- Gemini is not supported; the AI layer targets OpenAI-compatible APIs.
