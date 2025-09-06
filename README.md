# Optimal-Account-Balancing-Web-App
Optimum Account Balancing — Web App

Split. Track. Settle in minimum moves.
A web app that computes the least number of transactions required to balance shared expenses among a group (a.k.a. the “minimum cash-flow” problem).

✨ Features

Smart settlement engine (minimum number of transfers)

Multiple groups & currencies (INR, USD, …)

Per-expense splits (equal, shares, percentages, exact)

Audit trail & immutable logs

Invite & roles (owner, admin, member)

Export (CSV/PDF) and shareable settlement links

PWA (installable, offline-first reads)

Accessibility (WCAG AA), dark mode

🧠 How It Works (Algorithm)

We model each user’s net balance (amount paid − amount owed).
Greedy matching of max creditor with max debtor iteratively minimizes transfers.

1) Compute net[i] for each member
2) While max(net) > 0 and min(net) < 0:
     a) c = argmax(net)    (largest creditor)
     b) d = argmin(net)    (largest debtor)
     c) x = min(net[c], -net[d])
     d) record transfer d -> c of x
     e) net[c] -= x; net[d] += x
3) Stop when all |net[i]| < ε


This produces an optimal (minimum-edge) settlement for complete graphs of debt relations in O(n log n) with a heap; exact optimality on transaction count follows from repeatedly eliminating an extreme.
For sparse constraints (blocked pairs), we fall back to min-cost max-flow.

🧱 Tech Stack

Frontend: React + Vite, TypeScript, Tailwind, Zustand/Redux, React Query

Backend: Node.js + Express (or NestJS)

DB: PostgreSQL (Prisma ORM)
Alt: MongoDB/Mongoose or Firebase

Auth: JWT + OAuth (Google) via Passport.js

Cache/Queues: Redis

Infra: Docker, GitHub Actions, Fly.io/Render/Vercel

You can swap components; the app is modular.

📦 Repository Structure
.
├── apps/
│   ├── web/                 # React app
│   └── api/                 # Express/Nest API
├── packages/
│   ├── algo/                # Settlement engine (pure TS)
│   ├── ui/                  # Design system components
│   └── config/              # Shared ESLint, TS, Prettier
├── prisma/                  # DB schema & migrations
├── docker/                  # Docker compose & env samples
└── README.md

🗄️ Data Model (PostgreSQL/Prisma)
model User {
  id         String   @id @default(cuid())
  email      String   @unique
  name       String?
  pictureUrl String?
  members    Member[]
  createdAt  DateTime @default(now())
}

model Group {
  id          String   @id @default(cuid())
  name        String
  currency    String   @default("INR")
  members     Member[]
  expenses    Expense[]
  settlements Settlement[]
  createdAt   DateTime @default(now())
}

model Member {
  id       String @id @default(cuid())
  role     String @default("member") // owner/admin/member
  userId   String
  groupId  String
  user     User   @relation(fields: [userId], references: [id])
  group    Group  @relation(fields: [groupId], references: [id])
}

model Expense {
  id        String   @id @default(cuid())
  groupId   String
  paidById  String   // Member who paid
  title     String
  amount    Decimal  @db.Decimal(12,2)
  splits    Json     // [{memberId, shareType, value}]
  createdAt DateTime @default(now())
  group     Group    @relation(fields: [groupId], references: [id])
  paidBy    Member   @relation(fields: [paidById], references: [id])
}

model Settlement {
  id        String   @id @default(cuid())
  groupId   String
  fromId    String   // debtor member
  toId      String   // creditor member
  amount    Decimal  @db.Decimal(12,2)
  createdAt DateTime @default(now())
}

🔌 API (Express)
POST   /auth/login
GET    /groups
POST   /groups
POST   /groups/:id/invite
GET    /groups/:id/expenses
POST   /groups/:id/expenses
GET    /groups/:id/ledger           # computed balances
POST   /groups/:id/settlements/run  # compute optimal transfers
GET    /groups/:id/settlements


Settlement handler (sketch):

import { computeOptimalSettlements } from "@optimum/algo";

app.post("/groups/:id/settlements/run", async (req, res) => {
  const ledger = await buildLedger(req.params.id); // net balance per member
  const transfers = computeOptimalSettlements(ledger);
  await save(transfers);
  res.json({ transfers });
});

🚀 Getting Started
Prerequisites

Node 18+, pnpm 9+

Docker (optional)

PostgreSQL 14+

1) Clone & Install
git clone https://github.com/<you>/optimum-account-balancing.git
cd optimum-account-balancing
pnpm install

2) Environment

Create .env (see .env.example):

DATABASE_URL="postgresql://user:pass@localhost:5432/optimum"
JWT_SECRET="replace-me"
GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""

3) Database
pnpm prisma migrate dev
pnpm prisma db seed

4) Run (Dev)
# start API and Web concurrently
pnpm dev
# web: http://localhost:5173  api: http://localhost:3000

5) Docker (All-in-one)
docker compose up --build

🧪 Testing
pnpm test             # unit tests (algo + API)
pnpm test:watch
pnpm lint && pnpm typecheck


Property-based tests ensure the settlement engine conserves totals and terminates.

🛡️ Security & Privacy

JWT access & refresh tokens, short TTLs

Row-level authorization (group membership enforced at query)

Input validation (zod/express-validator)

Rate limiting & CORS

Minimal PII; delete on request

📈 Performance

Settlement engine: O(n log n) using heaps

Batched DB writes; prepared statements

Client-side caching (React Query) + ETags

Lazy-loaded routes & code-splitting

🗺️ Roadmap

Bank UPI/PayPal deep links on transfers

Multi-currency with FX snapshots

Per-category budgets & analytics

Custom constraints (avoid-person, caps) via min-cost flow

Real-time presence with WebSockets

Mobile app (React Native)

🤝 Contributing

Fork and create a feature branch

Add tests for your changes

Run pnpm lint && pnpm typecheck && pnpm test

Open a PR with a clear description

We follow Conventional Commits and keep PRs small & focused.

📜 License

MIT © You — see LICENSE

📷 Screenshots / Demo

Dashboard: Add image here

Add Expense: Add image here

Optimal Settlement: Add image here

(Optional) Publish a short Loom and link it here.

🧰 FAQ

Is the algorithm truly optimal?
Yes, for unconstrained settlements over a fully connected group, the greedy extreme-match algorithm yields a minimum number of transactions. With constraints, we switch to min-cost max-flow to respect edge limits.

Can I use Firebase instead of Postgres?
Yes—swap the DB layer; the @optimum/algo package is pure TypeScript.

How are fractions handled?
All money uses fixed-precision decimals (no floating errors). Currency rounding rules are applied at presentation.

🏷️ Badges (optional)

Add these to the top once set up:

[![CI](https://github.com/<you>/<repo>/actions/workflows/ci.yml/badge.svg)](…)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Quick Start Script (one-liner)
cp .env.example .env && pnpm install && pnpm prisma migrate dev && pnpm dev

