# Food Flight Tracker

**Track my Food — vom Feld bis ins Verkaufsregal.**

A consumer-facing platform that lets you scan a food product's barcode and instantly see a trust score for that specific batch, along with its full journey from farm to shelf. Think Yuka, but for the supply chain.

Built in ~24 hours at a hackathon organized by [Autexis](https://www.autexis.com/) (Swiss automation company). **Placed in Top 10 out of 30 teams.**

---

## What It Does

Scan any food barcode and get:

- **Trust Score (0–100)** for the exact batch — not the brand, but this specific LOT. Scored across cold chain integrity, quality inspections, time to shelf, producer track record, and handling steps.
- **Supply Chain Journey** — every step from harvest to retail, plotted on an interactive map with a timeline.
- **Recall Alerts** — if the batch is recalled, the app flags it immediately with severity, reason, and consumer instructions.
- **AI Chat Assistant ("Foodie")** — ask questions about the product's safety, origin, sustainability, or anomalies in natural language.
- **Anomaly Detection** — statistical flagging (z-score) when batch metrics deviate significantly from category averages.
- **Sustainability Metrics** — CO₂ footprint, water usage, and transport distance.
- **Leaderboard** — ranking of the most trustworthy producers.

## Architecture

| Layer | Tech | Repo |
|-------|------|------|
| **Backend API** | Go · chi router · pgx (raw SQL) · PostgreSQL | [`backend`](https://github.com/TrackMyFood/TrackMyFood-backend) |
| **Mobile App** | React Native · Expo · TypeScript | [`frontend`](https://github.com/TrackMyFood/TrackMyFood-frontend) |

The backend exposes a clean REST API. The single most important endpoint is `GET /api/scan/{barcode}` — it returns everything the app needs in one call: product info, batch data, pre-calculated trust score with sub-score breakdown, journey steps, certifications, recall status, sustainability data, and anomalies. No waterfalls.

### Trust Score Formula

```
TrustScore = 0.30 × ColdChain + 0.25 × QualityChecks + 0.20 × TimeToShelf
           + 0.15 × ProducerTrackRecord + 0.10 × HandlingSteps
```

If a sub-score has no data, its weight is redistributed proportionally. An active recall overrides the score to 0.

### Key API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/scan/{barcode}` | Full scan — product, batch, trust score, journey, recall, certs, sustainability |
| `GET` | `/api/batch/{id}/temperature` | Cold chain time-series data |
| `POST` | `/api/complaints` | File a consumer complaint (triggers score recalc) |
| `POST` | `/api/admin/recalls` | Create a recall (zeros score, returns affected users) |
| `POST` | `/api/auth/register` | User registration |
| `POST` | `/api/auth/login` | Authentication (JWT) |
| `GET` | `/api/leaderboard` | Top-scored producers |
| `POST` | `/api/scan/{barcode}/chat` | AI-powered Q&A about a product |

## Demo Scenarios

The backend is seeded with four products for live demo:

| Barcode | Product | Score | Scenario |
|---------|---------|-------|----------|
| `7610000000001` | Organic Strawberries | ~94 | Perfect batch — short journey, all checks passed |
| `7610000000002` | Atlantic Salmon | ~52 | Cold chain breach (12°C spike), failed quality check |
| `7610000000003` | Natural Yogurt | 0 | Active recall — contamination |
| `7610000000004` | Mountain Flower Honey | ~88 | Sustainable choice — Bio + Fair Trade certified |

## Running Locally

### Backend

```bash
# Clone and start (Docker)
git clone https://github.com/TrackMyFood/backend.git
cd backend
cp .env.example .env
make up        # starts PostgreSQL + API
make seed      # loads demo data
```

API runs on `http://localhost:8090`.

### Frontend

```bash
git clone https://github.com/TrackMyFood/frontend.git
cd frontend
npm install
npx expo start
```

Scan in Expo Go or run on a simulator.

---

*Built at the Autexis "Track my Food" Baden Hackt 2026, Switzerland.*
