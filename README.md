# AgriShare

> Rent the machine. Skip the purchase.

AgriShare is a farm equipment rental platform. Machine owners list tractors, harvesters, sprayers, and other equipment. Farmers can search for the machine they need, compare options by price, distance, and rating, and book it for the hours they need.

The platform runs as a responsive website and an installable Progressive Web App (PWA) from one codebase. It supports six Indian languages and is built with free, open-source tools.

## Table of Contents

- [The Problem](#the-problem)
- [Vision](#vision)
- [Who It's For](#who-its-for)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [DSA: The Algorithms Behind AgriShare](#dsa-the-algorithms-behind-agrishare)
- [Money Handling](#money-handling)
- [Penalty Rules](#penalty-rules)
- [Languages](#languages)
- [Project Structure](#project-structure)
- [Setup](#setup)
- [Install as an App](#install-as-an-app)
- [Deployment](#deployment-free-tiers)
- [Known Limitations](#known-limitations)
- [Future Scope](#future-scope)

## The Problem

- A new tractor costs roughly ₹6–10 lakh. Most small farmers cannot buy one, and many cannot afford even a used one.
- Farmers without machines may pay high hourly rates to whoever has one, or delay sowing and harvesting.
- Machine owners often leave their equipment idle for much of the year and have no easy way to earn from it.
- Government Custom Hiring Centres help, but they are offline, limited to one location, and offer no way to search or compare options.

AgriShare connects farmers and machine owners with search, ranking, booking, and a payment system designed to protect both sides.

## Vision

Every farmer should be able to use the right machine at the right time without owning it. Every machine should earn for its owner while it is not in use. The platform should be simple enough for a farmer with a basic phone and smart enough to make the best choice visible.

## Who It's For

| Role | What they do |
| --- | --- |
| **Farmer** | Searches for machines, compares options, books time slots, tops up a wallet, and rates owners. |
| **Machine owner** | Lists machines with photos and capabilities, sets prices and availability, accepts bookings, marks handover and return, and earns income. |
| **Admin** | Monitors users and bookings, suspends accounts, and reviews penalties. |

## Key Features

### For farmers

- Search by machine name with autocomplete.
- Filter by category, distance radius, date, and time.
- Compare three ranked views: **Cheapest**, **Nearest**, and **Best Balance**.
- Adjust how much price, distance, and rating matter with a slider and live re-ranking.
- View search results on a map.
- See a clear price breakdown for rent, refundable deposit, and late-return rules.
- Follow a booking timeline and preview refunds when cancelling.
- Use a wallet with top-up and transaction history.

### For machine owners

- Create machine profiles with up to six photos, capabilities, brand, horsepower, and a location pin.
- Manage an availability calendar and block time windows.
- Handle incoming requests, handovers, returns, and damage reports.
- View active bookings, monthly earnings, and utilization on a dashboard.
- Review an earnings ledger in plain language.

### For admins

- View overview statistics.
- Search users and suspend or unsuspend accounts.
- Review penalty and warning logs.

### Platform features

- **Late returns:** Charge extra per late hour, issue a warning for the first offense, and suspend for seven days after three late returns within 30 days.
- **Damage reports:** Tie reports to the relevant booking record.
- **Trust score:** Update scores from ratings after completed bookings.
- **Conflict-free bookings:** Prevent two farmers from booking the same machine for overlapping times.
- **Six languages:** English, Hindi, Kannada, Tamil, Telugu, and Marathi.
- **Light and dark themes.**
- **Installable PWA:** Supports Android and iPhone, with an offline shell.

## Tech Stack

The tools below are free and open source. Hosting uses free tiers.

| Layer | Technology | Why |
| --- | --- | --- |
| Frontend | React 18, Vite, TypeScript | Fast builds and typed components |
| Styling | Tailwind CSS, Framer Motion, Lucide Icons | Consistent design, subtle motion, and one icon set |
| Routing | React Router v6 | Role-based route guards |
| Server state | TanStack Query | Caching and invalidation for API data |
| Client state | Zustand | Lightweight auth, language, and theme state |
| Localization | i18next, react-i18next | Six languages with English fallback |
| Maps | Leaflet with OpenStreetMap | No API key required |
| PWA | vite-plugin-pwa | Installable app and offline shell |
| Backend | Node.js 20, Express, TypeScript | Mature, typed, and widely used |
| Database | PostgreSQL 16 | Reliable relational data and transactions |
| ORM | Prisma | Typed queries and migrations |
| Auth | JWT, bcryptjs | Stateless sessions and password hashing |
| Validation | Zod | Consistent request validation |
| Uploads | Multer, Sharp | Upload handling and image compression to WebP |
| Scheduler | node-cron | Late-return detection and cleanup |
| Security | Helmet, CORS, express-rate-limit | Standard hardening for Express |
| Logging | Pino | Fast structured logs |
| Local development database | Docker Compose | One command to start PostgreSQL |
| Free hosting options | Render or Railway (server), Vercel or Netlify (client), Neon or Supabase (database) | Free tiers for a student project |

### Distance and payments

- **Routing distance:** Uses the Haversine formula for straight-line distance and Dijkstra's algorithm over a location graph for road-like distance between hub nodes. An optional public OSRM demo server can be used, with Haversine as a fallback.
- **Payments:** Uses a double-entry internal wallet. A `PaymentProvider` interface allows a real gateway to be added later. The project uses a mock provider, so no real money moves.

## Architecture

```text
Browser / Installed PWA
        │  HTTPS (JSON, multipart)
        ▼
Express API
  middleware: helmet · cors · rate limit · auth · role · validate · error
        │
        ├── modules/*   routes → controller → service → Prisma
        │        │
        │        └── dsa/*   pure algorithms, no framework imports
        │
        ├── jobs/*      node-cron: late returns, cleanup
        │
        ▼
PostgreSQL (Prisma)            Local disk: uploads/
```

### Design rules

- `dsa/` imports nothing from Express, Prisma, or React. Its algorithms stand alone and can be explained in the project report.
- Every module follows the same path: route → controller → service → database. Only services touch Prisma.
- All money movements go through the wallet module inside a database transaction. No other module changes balances.
- The client calls the API only through `services/`. Pages and components never call `fetch` directly.
- `shared/` holds roles, statuses, categories, and penalty values used by both client and server.

## DSA: The Algorithms Behind AgriShare

Data structures and algorithms power real platform features.

### 1. Trie (prefix tree): machine name autocomplete

- **Where:** `GET /search/suggest`, used by the search bar dropdown.
- **Why:** Returns matching machine names as a farmer types, without scanning every machine.
- **Complexity:** Insert `O(L)`; prefix search `O(P + K)`, where `L` is word length, `P` is prefix length, and `K` is the number of results returned. Space: `O(total characters)`.
- **Example:** Typing `"tra"` returns “Tractor 45 HP”, “Trailer”, and similar matches in a few microseconds.

### 2. Dijkstra's shortest path: distance through the location graph

- **Where:** Road distance between a farmer's hub and a machine's hub.
- **Why:** Villages and hubs form a graph with weighted roads. Dijkstra finds the shortest route rather than only the straight-line distance.
- **Complexity:** `O((V + E) log V)` with a binary heap.
- **Example:** The path from Village A to Hub C through B costs 6.2 km, shorter than the direct 7.5 km edge.

### 3. Max-heap / priority queue: top-K ranking

- **Where:** Building the Cheapest, Nearest, and Best Balance views.
- **Why:** Returns the best `K` machines without sorting all results.
- **Complexity:** Push and pop: `O(log n)`. Top-K selection: `O(n log K)`.
- **Example:** From 500 matching machines, the top 10 are selected in one pass with a heap of size 10.

### 4. Interval index with binary search: booking conflict detection

- **Where:** Booking creation and search filtering.
- **Why:** Checks whether a requested time window overlaps an existing booking. Intervals are half-open, `[start, end)`, so a booking ending at 10:00 does not conflict with one starting at 10:00.
- **Complexity:** Overlap check `O(log n + k)`, where `k` is the number of overlapping intervals found. Insert: `O(n)` worst case for array shifting.
- **Example:** If a booking exists from 9:00–11:00, a request for 10:30–12:00 is rejected, while a request for 11:00–12:00 is accepted.

### 5. FIFO queue: fair booking order

- **Where:** Per-machine booking requests.
- **Why:** When two farmers request the same machine at the same moment, the first request is processed first.
- **Complexity:** Enqueue and dequeue: `O(1)`.

### 6. Hash maps: instant lookups

- **Where:** Machine, user, and booking lookups by ID, and per-user penalty counts.
- **Complexity:** Average `O(1)` for get and set.

### 7. Min-max normalization and weighted scoring: the ranking formula

- **Where:** The Best Balance view and the weight slider.
- **Why:** Price (rupees), distance (km), and rating (1–5) use different units. Normalization puts them on one 0-to-1 scale before combining.

```text
normalized(x) = (x - min) / (max - min)       // if max = min, use 0

score = wPrice × (1 - normPrice)
      + wDistance × (1 - normDistance)
      + wRating × normRating
```

Lower price and distance score higher; higher rating scores higher. Weights are normalized to sum to 1.

- **Complexity:** `O(n)` to normalize and score.

### 8. Haversine formula: straight-line distance

- **Where:** Distance when no graph route applies, and for map sorting.
- **Complexity:** `O(1)` per pair.

### Complexity summary

| Function | Time | Space |
| --- | --- | --- |
| Trie insert | `O(L)` | `O(total characters)` |
| Trie prefix search | `O(P + K)` | `O(K)` |
| Heap push/pop | `O(log n)` | `O(n)` |
| Top-K selection | `O(n log K)` | `O(K)` |
| Dijkstra | `O((V + E) log V)` | `O(V)` |
| Interval overlap check | `O(log n + k)` | `O(1)` |
| FIFO enqueue/dequeue | `O(1)` | `O(n)` |
| Min-max normalization | `O(n)` | `O(n)` |
| Haversine | `O(1)` | `O(1)` |

## Money Handling

- All amounts are stored as integer paise (`₹1 = 100 paise`). No floating-point money is used.
- The wallet uses double-entry bookkeeping. Every movement creates a debit and a matching credit.
- Balances cannot go negative. The check runs inside the same transaction as the debit.
- Deposits are held during a booking and refunded or forfeited when the machine is returned.

## Penalty Rules

| Event | Result |
| --- | --- |
| Return after booking end time | Late fee of 1.5× the hourly rate per started hour late |
| First late return | Warning |
| Three late returns within 30 days | 7-day suspension |
| Damage reported by owner | Damage penalty set by owner with a note, linked to the booking |
| Cancellation more than 24 hours before start | Full refund |
| Cancellation within 24 hours of start | 50% refund |
| Suspended user | Cannot create bookings or list machines |

## Languages

| Code | Language |
| --- | --- |
| `en` | English |
| `hi` | Hindi |
| `kn` | Kannada |
| `ta` | Tamil |
| `te` | Telugu |
| `mr` | Marathi |

The server returns error codes, and the client translates them. Missing keys fall back to English. To add a language, create `client/public/locales/{code}/translation.json`, add the code to `shared/constants.ts`, and add the font to `globals.css`.

## Project Structure

```text
agrishare/
├── client/          React + Vite + TypeScript frontend (website + PWA)
├── server/          Express + Prisma + TypeScript backend
│   ├── prisma/      Schema, migrations, seed
│   └── src/
│       ├── dsa/     Pure algorithm engine
│       ├── modules/ Feature modules (auth, machines, search, bookings, wallet, penalties, ratings, admin)
│       ├── jobs/    Scheduled tasks
│       └── middleware/, config/, utils/
├── shared/          Constants and types used by client and server
├── docs/            DECISIONS.md, DSA_NOTES.md, API.md
├── docker-compose.yml
└── .env.example
```

## Setup

### Requirements

- Node.js 20+
- Docker
- Git

### Run locally

```bash
# 1. Clone and enter the project
git clone <repo-url> agrishare
cd agrishare

# 2. Create environment files
cp .env.example .env
cp .env.example server/.env

# 3. Start PostgreSQL
docker compose up -d

# 4. Install and start the server
cd server
npm install
npx prisma migrate dev
npm run seed
npm run dev
```

In a second terminal, start the client:

```bash
cd client
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173).

### Demo accounts

These accounts are created by the seed:

| Role | Phone | Password |
| --- | --- | --- |
| Owner | `9000000001` | `password123` |
| Farmer | `9000000011` | `password123` |
| Admin | `9000000021` | `password123` |

## Install as an App

- **Android (Chrome):** Open the site, tap the menu, then choose **Install app** or **Add to Home screen**.
- **iPhone (Safari):** Open the site, tap the Share button, then choose **Add to Home Screen**.

## Deployment (Free Tiers)

1. **Database:** Create a free Neon or Supabase PostgreSQL instance. Set its connection string as `DATABASE_URL`.
2. **Server:** Deploy `server/` to Render or Railway. Set the environment variables from `.env.example`. Use `npm run build` as the build command and `npm start` as the start command.
3. **Client:** Deploy `client/` to Vercel or Netlify. Set `VITE_API_URL` to the server URL.
4. **CORS:** Set `CLIENT_URL` on the server to the deployed client URL.

## Known Limitations

- Payments are simulated; no real money moves.
- Uploaded images are stored on local disk. For production, move them to object storage.
- Road distance uses a demo graph of hubs. Coverage grows as more locations are added.
- Notifications are in-app only. SMS and push notifications are planned.

## Future Scope

- Native mobile builds
- Real payment gateway
- GPS tracking for machines
- Crop-insurance linkage
- Voice search in local languages
- Owner-side maintenance reminders
