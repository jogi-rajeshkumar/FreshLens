<div align="center">

<img src="https://img.shields.io/badge/Status-In%20Development-00BFA6?style=for-the-badge&logo=github" />
<img src="https://img.shields.io/badge/Duration-90%20Days-3B82F6?style=for-the-badge" />
<img src="https://img.shields.io/badge/Stack-Python%20%7C%20React%20Native-8B5CF6?style=for-the-badge" />
<img src="https://img.shields.io/badge/Collaborators-Welcome-22C55E?style=for-the-badge" />

<br /><br />

# 🔍 FreshLens
### AI Fridge Food Quality Tracker

> **Snap your fridge. Know what's fresh. Waste nothing.**

*Computer vision meets everyday life — detect food freshness in real time, track items over time, and get intelligent meal suggestions before anything expires.*

<br />

**[📋 View Full Project Plan](#-project-plan) · [🏗️ Architecture](#️-system-architecture) · [🚀 Getting Started](#-getting-started) · [🤝 Collaborate With Me](#-collaborate-with-me)**

</div>

---

## 🧠 The Idea Behind FreshLens

I've been obsessed with a simple problem: **we throw away a lot of food, and most of it is completely avoidable.**

Most people don't track what's in their fridge. They forget about the yogurt at the back, the bell peppers going soft, the leftovers from three days ago. There are apps for shopping lists and meal planning — but nothing that looks at your actual fridge and tells you *right now* what's about to go bad.

That's FreshLens.

You open the app, point your camera at your fridge shelf, and within seconds you get:
- Every food item detected and labelled
- A freshness score for each one: **Fresh 🟢**, **Borderline 🟡**, or **Spoiled 🔴**
- A running pantry that tracks how each item changes day by day
- Meal suggestions powered by an LLM, prioritising whatever is about to expire
- Push alerts before things go bad — not after

The backend is a fully universal REST API built in Python/FastAPI, so any frontend (mobile, web, IoT camera) can consume the same endpoints. The mobile app is React Native with Expo. The models are YOLOv8n for detection and EfficientNet-B3 for freshness classification, both exported to ONNX for fast, portable inference.

I'm building this as a 90-day end-to-end project — from raw dataset collection to a live, deployed, production API. Every phase is documented here.

---

## ✨ Features

| Feature | Description |
|---|---|
| 📸 **Fridge Scan** | Upload or capture a fridge photo; YOLOv8n detects every item with bounding boxes |
| 🎯 **Freshness Grading** | EfficientNet-B3 classifies each item: Fresh / Borderline / Spoiled |
| 📦 **Pantry Tracker** | Items are tracked across multiple scans; freshness history plotted as a timeline |
| 🍽️ **Smart Meal Suggestions** | Claude API generates 3 ranked recipes using items expiring soonest |
| 📊 **Waste Analytics** | Track your waste rate, money saved estimate, and most-wasted food categories |
| 🔔 **Push Alerts** | FCM notifications when items have ≤ 2 days remaining |
| 🌐 **Universal API** | Same REST endpoints work for mobile, web, or any IoT device |
| 📴 **Offline Mode** | Cached pantry visible when offline; scans queue and sync on reconnect |

---

## 🗺️ Project Plan

This is a **90-day full build** structured across 5 phases. Every single day has a defined task, expected output, and success criterion. No hand-waving.

```
Days  1–18  │ Phase 1 │ Research & Data Collection
Days 19–45  │ Phase 2 │ Backend (Python / FastAPI)
Days 46–72  │ Phase 3 │ Frontend (React Native / Expo)
Days 73–85  │ Phase 4 │ Testing (unit → load → user study)
Days 86–90  │ Phase 5 │ Deployment (Docker → AWS → CI/CD)
```

### Phase 1 — Research & Data Collection (Days 1–18)

The thesis here is simple: **garbage in, garbage out.** No model architecture saves you from bad training data. I'm not skipping this phase.

- **Days 1–2:** Literature review — YOLOv8 paper, EfficientNet, FoodNet, PatchCore, freshness grading surveys. Write a 1-page synthesis.
- **Days 3–4:** Dataset audit — Food-101, Open Images food subset, Kaggle Fresh & Rotten, FreshNet. Log class counts, identify gaps.
- **Days 5–8:** Custom photo collection — fridge photo rig with diffuse lighting, 3 angles, 4 freshness stages per item. Targeting 1,000+ raw images.
- **Days 9–12:** Label Studio annotation — bounding boxes + 3-class freshness labels. Target: 5,000+ annotated images across 50 food categories.
- **Days 13–18:** Baseline models — YOLOv8n fine-tune + EfficientNet-B3 classifier. Establish mAP50 and accuracy benchmarks.

**Exit criteria:** YOLOv8n mAP50 > 0.78 · EfficientNet accuracy > 82% · 5,000+ annotated images · augmentation pipeline producing 3× effective dataset size.

---

### Phase 2 — Backend Development (Days 19–45)

Python 3.11, FastAPI, PostgreSQL, Redis, ONNX Runtime. Every endpoint designed contract-first in OpenAPI before any code is written.

**Folder structure:**
```
freshlens-backend/
├── app/
│   ├── main.py              # FastAPI app, CORS, routers
│   ├── config.py            # pydantic-settings
│   ├── database.py          # SQLAlchemy engine + session
│   ├── routers/
│   │   ├── auth.py          # /auth endpoints
│   │   ├── scan.py          # POST /scan
│   │   ├── pantry.py        # GET /pantry, /items
│   │   ├── meals.py         # POST /meals
│   │   └── analytics.py     # GET /analytics
│   ├── models/
│   │   └── db.py            # SQLAlchemy ORM models
│   ├── schemas/             # Pydantic request/response schemas
│   └── services/
│       ├── inference.py     # ONNX model loading + inference
│       ├── tracker.py       # Item ID consistency across scans
│       ├── meal_service.py  # Claude API call + parsing
│       ├── decay_model.py   # Freshness decay estimation
│       └── notification.py  # FCM push sender
├── models/                  # ONNX model files
├── tests/                   # pytest suite
├── alembic/                 # DB migrations
├── Dockerfile
├── docker-compose.yml
└── requirements.txt
```

**API endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/auth/register` | Register → returns `user_id` + JWT |
| `POST` | `/api/v1/auth/login` | Login → `access_token` (15min) + `refresh_token` (7d) |
| `POST` | `/api/v1/scan` | Upload fridge image → detected items + freshness scores |
| `GET`  | `/api/v1/pantry` | All tracked items with `freshness_status` + `days_remaining` |
| `GET`  | `/api/v1/items/{id}` | Single item: label, category, full freshness history |
| `GET`  | `/api/v1/items/{id}/history` | Time-series `[{timestamp, score}]` for chart rendering |
| `POST` | `/api/v1/meals` | LLM call with pantry context → 3 ranked meal suggestions |
| `GET`  | `/api/v1/analytics` | Waste rate %, money saved estimate, most-wasted category |
| `DELETE` | `/api/v1/items/{id}` | Remove item manually (used or discarded) |

---

### Phase 3 — Frontend (React Native / Expo) (Days 46–72)

Eight screens. Every one wire-framed before a line of code. The design system is built around the freshness signal: green for fresh, amber for borderline, red for spoiled — consistent everywhere.

**Screens:** Auth → Camera → Scan Results → Pantry List → Item Detail → Meals → Analytics → Notifications

Key integrations:
- `react-native-vision-camera` for full-screen camera capture
- `react-native-svg` for bounding box overlay on scan results
- `react-native-chart-kit` for freshness history timeline
- `Zustand` + `AsyncStorage` for state + offline persistence
- `SecureStore` for JWT storage
- `NetInfo` for offline detection + scan queue

---

### Phase 4 — Testing (Days 73–85)

No hand-wavy "it seems to work." Real tests, real numbers.

- **Unit tests (Days 73–75):** pytest suite with fixtures — mock ONNX, test DB, test user. All endpoints tested happy path + error cases.
- **Integration tests (Days 76–78):** Full scan flow with real models + DB. 5-day pantry drift simulation. Auth flow: register → login → refresh → logout. Rate limit trigger tests.
- **Load tests (Days 79–80):** Locust with 50 concurrent users. P95 `/scan` target < 2s. ONNX inference profiling — each stage must be < 300ms on CPU.
- **User study (Days 81–83):** 10 participants, 5 fridge scenarios (fresh / mixed / nearly empty / single item / poor lighting). Think-aloud protocol, freshness agreement measured.
- **Bug fix sprint + security review (Days 84–85):** SQL injection, JWT secret strength, rate limits on all write endpoints, image path traversal prevention.

**Exit criteria:** 80%+ pytest coverage · P95 < 2s under load · freshness prediction agreement > 75% · zero SQL injection / path traversal vulnerabilities.

---

### Phase 5 — Deployment (Days 86–90)

Five days. One production system.

- **Day 86:** Multi-stage Dockerfile. `python:3.11-slim` base. `.dockerignore`.
- **Day 87:** `docker-compose.yml` — `api + postgres + redis`. Health checks. `restart: unless-stopped`.
- **Day 88:** Deploy to AWS EC2 `t3.medium` (or Render). Push image to ECR / Docker Hub.
- **Day 89:** Nginx reverse proxy → FastAPI :8000. Certbot + Let's Encrypt. HTTP → HTTPS redirect.
- **Day 90:** GitHub Actions CI/CD: push to main → pytest → Docker build → push to ECR → SSH deploy. Sentry DSN added. UptimeRobot monitoring live.

**Exit criteria:** Live public HTTPS URL · CI/CD running on every push · Sentry grouping errors · UptimeRobot monitoring · app distributed via Expo Go / TestFlight.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────┐
│              CLIENT LAYER                        │
│   📱 React Native App  🌐 Web  🔌 IoT Camera    │
└──────────────────┬──────────────────────────────┘
                   │ HTTPS / REST
┌──────────────────▼──────────────────────────────┐
│         🔒 Nginx + JWT Auth Gateway              │
│         Rate Limiting · CORS · SSL               │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│           ⚡ FastAPI Application                 │
│  auth · scan · pantry · meals · analytics        │
└────┬──────────┬────────────┬────────────┬───────┘
     │          │            │            │
┌────▼───┐ ┌───▼────┐ ┌─────▼──┐ ┌──────▼──┐
│  ONNX  │ │Tracker │ │  LLM   │ │  Push   │
│Infer.  │ │Service │ │ Meals  │ │ Notify  │
│YOLOv8n │ │ID Map  │ │Claude  │ │FCM/Sch. │
│+ENet   │ │        │ │  API   │ │         │
└────┬───┘ └────────┘ └────────┘ └─────────┘
     │
┌────▼──────────────────────────────────────┐
│  🗄️ PostgreSQL  ⚡ Redis  📁 S3/Storage   │
│  Items·Scans   Sessions  Image files      │
└───────────────────────────────────────────┘

                 ML PIPELINE
Raw Photos → Label Studio → Albumentations
     → YOLOv8n Train (mAP50 > 0.78)
     → EfficientNet-B3 Train (Acc > 82%)
     → ONNX Export → Deployed in API
```

---

## 🛠️ Tech Stack

| Layer | Technologies |
|---|---|
| **CV Models** | YOLOv8n · EfficientNet-B3 · ONNX Runtime |
| **Backend** | Python 3.11 · FastAPI · SQLAlchemy · Alembic · PostgreSQL · Redis · APScheduler |
| **AI / ML** | PyTorch · Ultralytics · Albumentations · Weights & Biases · Label Studio |
| **Mobile** | React Native · Expo · react-native-vision-camera · react-native-onnxruntime · Zustand |
| **DevOps** | Docker · docker-compose · Nginx · AWS EC2 · GitHub Actions · Sentry · Certbot |
| **Integrations** | Claude API (meal suggestions) · Firebase Cloud Messaging · USDA FoodData API |

---

## 🚀 Getting Started

### Prerequisites

```bash
python 3.11+
node 18+
docker & docker-compose
expo-cli
```

### Backend Setup

```bash
# Clone the repo
git clone https://github.com/rajeshkumarjogi/freshlens.git
cd freshlens/freshlens-backend

# Create virtual environment
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env with your PostgreSQL, Redis, Claude API, FCM credentials

# Run DB migrations
alembic upgrade head

# Start the API
uvicorn app.main:app --reload --port 8000
```

### With Docker (recommended)

```bash
docker-compose up --build
# API live at http://localhost:8000
# Docs at http://localhost:8000/docs
```

### Mobile App Setup

```bash
cd freshlens-mobile
npm install
npx expo start
# Scan QR code with Expo Go on your device
```

> ⚠️ **Note:** The backend API URL must be set in `app/config/api.ts` before running the mobile app. Point it to your local IP or deployed API endpoint.

---

## 🤝 Collaborate With Me

Here's the honest truth: I'm one person with a full vision, and there's a lot to build. I'm not looking for passive stars — I'm looking for **people who want to actually build something together.**

This project touches computer vision, mobile development, backend engineering, DevOps, and AI/LLM integration. There's room for contributions at every level, whether you're a student, a professional, or someone who just wants to learn by doing something real.

---

### 🌱 Where I'd love help

**🤖 Computer Vision**
- Help improve the annotation pipeline or dataset quality
- Experiment with model architectures beyond YOLOv8n — could MobileNet work? Lighter models for on-device inference?
- On-device inference with `react-native-onnxruntime` — this is tricky and I could use a collaborator who's done it before

**⚙️ Backend / API**
- Code review on the FastAPI architecture — is the service layer clean enough?
- Help writing the test suite (pytest, Locust) — if you enjoy writing tests, I genuinely want your eyes on this
- Database schema review — indexes, query optimisation, connection pooling

**📱 Mobile (React Native)**
- Anyone who's done `react-native-vision-camera` integration before — I know there are quirks
- UX feedback from real mobile developers — I'm building this for regular people, not engineers
- iOS / Android compatibility testing

**🚀 DevOps / Cloud**
- AWS architecture review — is `t3.medium` + RDS the right call, or is there a smarter setup?
- GitHub Actions pipeline review — I want this bulletproof
- Anyone who's deployed ML models with ONNX in production containers

**📊 Data / ML**
- More diverse food photography — especially non-Western fridge contents (my training data skews heavily European/American right now, and that's a real limitation I'm aware of)
- Freshness scoring calibration — I need people willing to label images and tell me when my model is wrong

---

### 💬 How to reach me

I'd genuinely love to talk if any of this interests you:

- **LinkedIn:** [Rajesh Kumar Jogi](https://www.linkedin.com/in/rajeshkumarjogi)
- **Email:** Open an issue or reach me through LinkedIn — I respond to everyone
- **GitHub Discussions:** I'll keep a Discussions board active for ideas, questions, and design decisions

If you want to contribute code, open a PR. If you want to talk first, open an issue or a Discussion. If you just want to follow along, watch the repo — I'll be pushing regularly and documenting the whole process.

---

### 🛠️ How to contribute (code)

```bash
# 1. Fork the repo
# 2. Create your feature branch
git checkout -b feature/your-feature-name

# 3. Commit your changes (please write meaningful commits)
git commit -m "feat: add freshness decay model calibration"

# 4. Push to your fork and open a PR
git push origin feature/your-feature-name
```

PRs should include:
- A clear description of *what* and *why*
- Tests for any new functionality (even simple ones)
- Any relevant screenshots or benchmark numbers if it's ML/CV work

I review every PR personally and I'll give detailed feedback, not just a thumbs up or thumbs down.

---

### 🎓 A note for students

I'm doing this as part of my MSc AI programme at the University of East London, alongside my work at Green Environment Ltd. I know exactly what it's like to be trying to build something real as a student — juggling coursework, not having access to expensive GPUs, trying to make a portfolio that stands out.

If you're a student who wants to learn by contributing to a real project, **please reach out.** I'm not gatekeeping this. You don't need to be an expert. You need to be curious, willing to learn, and honest about what you know and don't know. That's it.

---

## 📁 Repository Structure

```
freshlens/
├── freshlens-backend/          # Python / FastAPI API
│   ├── app/
│   ├── models/                 # ONNX model files
│   ├── tests/
│   ├── alembic/
│   ├── Dockerfile
│   └── docker-compose.yml
├── freshlens-mobile/           # React Native / Expo app
│   ├── app/
│   ├── components/
│   ├── stores/
│   └── api/
├── ml/                         # Training scripts & notebooks
│   ├── data/                   # Dataset management
│   ├── train_yolo.py
│   ├── train_efficientnet.py
│   ├── augment.py
│   └── export_onnx.py
├── docs/                       # Architecture diagrams, API specs
│   └── openapi.yaml
└── README.md
```

---

## 📊 Progress Tracker

| Phase | Status | Days | Key Milestone |
|-------|--------|------|---------------|
| Phase 1 — Research & Data | 🔄 In Progress | 1–18 | Annotated dataset + baseline models |
| Phase 2 — Backend | ⏳ Upcoming | 19–45 | Full REST API with all endpoints |
| Phase 3 — Frontend | ⏳ Upcoming | 46–72 | Working app: scan → pantry → meals |
| Phase 4 — Testing | ⏳ Upcoming | 73–85 | 80%+ coverage, user study done |
| Phase 5 — Deployment | ⏳ Upcoming | 86–90 | Live HTTPS API + CI/CD |

---

## 📄 License

MIT License — use it, fork it, build on it. If you do something interesting with it, I'd love to know.

---

<div align="center">

**Built by [Rajesh Kumar Jogi](https://www.linkedin.com/in/rajeshkumarjogi)**
*AI Agent Developer · MSc Artificial Intelligence — University of East London · Green Environment Ltd*

<br />

*If this project resonates with you — whether you want to contribute, collaborate, or just follow along — the door is open.*

⭐ Star the repo if you find it useful · 🍴 Fork it if you want to build something · 🤝 Open an issue if you want to talk

</div>
