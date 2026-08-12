# SafeHer 🆘

**A women's safety platform that turns "something feels wrong" into help arriving — in seconds.**

SafeHer unifies four pillars into one real-time response pipeline: instant SOS alerts, hazard-aware safe routing, passive acoustic detection, and anonymous incident reporting — all feeding the same backend so prevention, detection, response, and systemic change work together instead of as separate tools.

---

## ✨ Features

### 🆘 Hidden SOS Trigger
- Hold-to-activate trigger (with a 3–5s non-blocking "confirm you're safe" undo window)
- Fires **three independent alert channels in parallel** — WebSocket (speed), Twilio SMS/voice, and email — so one channel failing never blocks another
- Live location streams to trusted contacts and an observer dashboard until the incident is resolved
- Full delivery audit log per incident (`incident_alerts`) — every channel attempt is logged as `sent`, `delivered`, `failed`, or `simulated`

### 📍 Safe-Route Navigation
- Routes scored by **crowd-sourced hazard density**, not just distance — using MySQL's native `POINT` + `SPATIAL INDEX` + `ST_Distance_Sphere()` for geo queries (no PostGIS dependency)
- One-tap anonymous hazard pin reporting with category chips
- Nearby **police stations and hospitals** surfaced live via the Overpass API (OpenStreetMap) — free, no API key required
- Live location tracking that keeps route scoring current as you move

### 🎙 Acoustic Anomaly Detection
- Passive scream/distress detection that can auto-trigger the same SOS pipeline
- Audio processed on-device / in short buffered clips only — never streamed continuously, and only stored as evidence after an actual detection event

### 📝 Anonymous Reporting Portal
- No login required to submit a report
- **Zero identity collection** — no IP address, no device fingerprint stored, ever
- A random, non-sequential tracking token is the only way to check a report's status later
- Uploaded photos are automatically stripped of EXIF metadata server-side before storage

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python / FastAPI |
| Database | MySQL 8.0+ (SQLite fallback for local dev) |
| Realtime | FastAPI native WebSocket, in-process broadcast |
| SMS / Voice | Twilio |
| Email | SendGrid |
| Routing | Mapbox Directions API / self-hosted OSRM |
| Nearby places | Overpass API (OpenStreetMap) |
| Auth | JWT (PyJWT + bcrypt) |
| Frontend | HTML / CSS / JS (single-page) |

---

## 🚀 Getting Started

### Prerequisites
- Python 3.10+
- MySQL 8.0+ (optional — the app falls back to SQLite automatically if no MySQL connection is configured)

### 1. Clone and install dependencies
```bash
git clone https://github.com/nilanjannath722-dotcom/SafeHer24-7.git
cd SafeHer24-7
pip install -r requirements.txt
```

### 2. Configure environment variables
```bash
cp _env.example .env
```
Then fill in `.env`:

| Variable | Purpose | Required? |
|---|---|---|
| `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` | MySQL connection (leave blank to fall back to SQLite) | No |
| `SECRET_KEY` | JWT signing secret | Yes |
| `ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES` | JWT config | Yes (defaults provided) |
| `DEMO_MODE` | Simulates SMS sends instead of firing real ones (required for India — see note below) | Recommended `true` for demos |
| `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_PHONE_NUMBER` | SMS/voice alerts | Only for live SMS/voice |
| `SENDGRID_API_KEY`, `FROM_EMAIL` | Email alerts | Only for live email |
| `GOOGLE_MAPS_API_KEY` | Client-side maps | Optional |

> **India note:** Real commercial SMS to Indian numbers requires TRAI DLT registration (24–72h approval, not achievable in a hackathon window). Keep `DEMO_MODE=true` to simulate + log SMS sends. **Twilio voice calls do not require DLT** and will fire live even in demo mode — use voice for a real, live-landing alert during a demo.

### 3. Run the server
```bash
python run.py
```
The API will be available at `http://127.0.0.1:8000`.

### 4. Run tests
```bash
python -m pytest test_backend.py
```
The test suite covers auth, contacts, SOS trigger/resolve, hazard reporting + route scoring, anonymous reports with EXIF stripping, both WebSocket flows (victim + observer), and nearby-places lookups.

---

## 📁 Project Structure
```
SafeHer24-7/
├── backend/              # FastAPI application (routers, models, services)
├── index.html            # Frontend (single-page)
├── test_backend.py       # Full backend test suite
├── requirements.txt      # Python dependencies
├── run.py                # App entry point
├── _env.example           # Environment variable template
└── safeher.db            # Local SQLite dev database (auto-created)
```

---

## 🔌 Key API Endpoints

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/auth/register` / `/auth/login` | Account creation & login |
| `GET` `POST` `PUT` `DELETE` | `/contacts` | Manage emergency contacts |
| `POST` | `/sos` | Trigger an SOS incident |
| `WS` | `/sos/ws/{incident_id}` | Live incident tracking (observer) |
| `WS` | `/sos/ws` | Victim-side SOS trigger + live location stream |
| `POST` | `/incidents/{id}/resolve` | Resolve/cancel an active incident |
| `GET` | `/hazards/nearby` | Hazard reports near a point |
| `POST` | `/hazards/score-route` | Score candidate routes by hazard risk |
| `GET` | `/places/nearby?type=police\|hospital` | Nearby police stations / hospitals |
| `POST` | `/reports/anonymous` | Submit an anonymous report |
| `GET` | `/reports/status/{token}` | Check a report's status by tracking token |

---

## 🔒 Privacy & Reliability by Design

- **No single point of failure for SOS alerts** — WebSocket, SMS/voice, and email fire independently; one failing never blocks another.
- **Anonymous reports collect nothing identifying** — no IP, no device fingerprint, ever, enforced at the schema level, not just the UI.
- **Audio is never streamed continuously** — on-device processing, short buffered clips only, and only after an actual detection event.
- Design aligns with India's DPDP Act 2023 data-minimization principles.

---

## 🗺 Roadmap / Future Scope

- Real Aadhaar-linked identity verification (requires UIDAI AUA/KUA authorization)
- Production SMS via TRAI DLT-registered sender ID
- Live YAMNet/TFLite acoustic classification (currently demo-mode with pre-recorded clip fallback)
- India-region hosting (AWS `ap-south-1` / GCP `asia-south1`) for production latency


