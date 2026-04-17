# ⬡ NexRide AI — ML Ride Intelligence Platform
> Uber Michelangelo-inspired ML system for real-time ride demand prediction and dynamic surge pricing

![Python](https://img.shields.io/badge/Python-3.11-blue) ![FastAPI](https://img.shields.io/badge/FastAPI-0.111-green) ![scikit-learn](https://img.shields.io/badge/scikit--learn-1.4-orange) ![Docker](https://img.shields.io/badge/Docker-ready-blue)

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     NexRide AI Platform                      │
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  Data Layer  │───▶│  ML Models   │───▶│  FastAPI     │  │
│  │              │    │              │    │              │  │
│  │ preprocessing│    │ Random Forest│    │/predict-     │  │
│  │ 87,600 rows  │    │ (demand)     │    │ demand       │  │
│  │ 10 locations │    │ R²=0.986     │    │/predict-     │  │
│  │ 5 weather    │    │              │    │ price        │  │
│  │ 365 days     │    │ Grad.Boost   │    │/predict-all  │  │
│  │              │    │ (surge)      │    │              │  │
│  │              │    │ R²=0.975     │    │              │  │
│  └──────────────┘    └──────────────┘    └──────┬───────┘  │
│                                                 │          │
│  ┌──────────────┐    ┌──────────────┐           │          │
│  │  Simulator   │    │  Frontend    │◀──────────┘          │
│  │              │    │              │                      │
│  │ Real-time    │    │ Animated     │                      │
│  │ ride stream  │    │ React SPA    │                      │
│  │ →dashboard   │    │ Chart.js     │                      │
│  └──────────────┘    └──────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

## 📁 Project Structure

```
ride-ml-system/
├── backend/
│   ├── api/
│   │   └── main.py              # FastAPI app (5 endpoints)
│   ├── src/
│   │   ├── preprocessing.py     # Data gen + feature engineering
│   │   ├── feature_engineering.py # Feature store utilities
│   │   ├── train.py             # Model training pipeline
│   │   └── predict.py           # Inference utilities
│   ├── simulation/
│   │   └── simulator.py         # Real-time ride request simulator
│   ├── models/                  # Saved .pkl files (after training)
│   │   ├── demand_model.pkl     # Random Forest (R²=0.986)
│   │   ├── price_model.pkl      # Gradient Boosting (R²=0.975)
│   │   └── metrics.json         # Evaluation metrics
│   ├── data/                    # Raw + processed datasets
│   ├── dashboard/               # Simulator output (CSV + JSONL)
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/
│   └── index.html               # Full animated SPA (no build step)
├── docker-compose.yml
├── nginx.conf
└── README.md
```

## 🤖 ML Models

| Model | Algorithm | Target | R² | RMSE |
|-------|-----------|--------|-----|------|
| Demand | Random Forest (200 trees) | Rides/hour | 0.986 | 5.39 |
| Surge | Gradient Boosting (200 est.) | Surge multiplier | 0.975 | 0.10 |

**Top Features — Demand Model:**
- `driver_availability` (94%)
- `is_peak_hour`, `hour_cos`, `weather_Rainy`

**Top Features — Surge Model:**
- `weather_Stormy` (31%), `weather_Rainy` (20%)
- `is_peak_hour` (18%), `hour_cos` (7%)

## 🚀 Quick Start (Local)

### 1. Clone & Install
```bash
git clone https://github.com/YOUR_USERNAME/ride-ml-system.git
cd ride-ml-system/backend
pip install -r requirements.txt
```

### 2. Generate Data + Train Models
```bash
# Generates 87,600 rows of synthetic ride data
python src/preprocessing.py

# Trains both models (takes ~90s on CPU)
python src/train.py
```
Expected output:
```
Demand R²: 0.986  RMSE: 5.394
Surge  R²: 0.975  RMSE: 0.102
```

### 3. Start FastAPI Server
```bash
uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload
```
API docs: http://localhost:8000/docs

### 4. Open Frontend
```bash
# Just open in browser — no build step needed
open frontend/index.html
# Or serve with Python
python -m http.server 3000 --directory frontend
```

### 5. Run Real-time Simulator
```bash
# In a separate terminal (API must be running)
python simulation/simulator.py

# Config via env vars
API_URL=http://localhost:8000 INTERVAL_SECONDS=1 python simulation/simulator.py
```

## 🐳 Docker Deployment

### Build & Run Backend
```bash
cd backend
docker build -t nexride-backend .
docker run -d -p 8000:8000 --name nexride-api nexride-backend
```

### Full Stack with Docker Compose
```bash
# First build frontend (if using React build)
# Or just serve the static index.html directly

docker-compose up --build

# Backend:  http://localhost:8000
# Frontend: http://localhost:3000
```

## ☁️ AWS Deployment

### EC2 Backend Deployment
```bash
# 1. Launch EC2 (Ubuntu 22.04, t3.medium or better)
# 2. SSH in and install Docker
ssh -i your-key.pem ubuntu@YOUR_EC2_IP
sudo apt update && sudo apt install -y docker.io docker-compose
sudo usermod -aG docker ubuntu && newgrp docker

# 3. Clone and deploy
git clone https://github.com/YOUR_USERNAME/ride-ml-system.git
cd ride-ml-system/backend
docker build -t nexride-backend .
docker run -d -p 80:8000 --restart always nexride-backend

# 4. Open port 80 in EC2 Security Group (inbound rule: TCP 80 from 0.0.0.0/0)
```

### S3 for Model/Data Storage
```bash
# Create bucket
aws s3 mb s3://nexride-ml-artifacts --region us-east-1

# Upload trained models
aws s3 cp backend/models/ s3://nexride-ml-artifacts/models/ --recursive

# Upload processed data
aws s3 cp backend/data/ s3://nexride-ml-artifacts/data/ --recursive

# Required IAM policy (attach to EC2 role):
# s3:GetObject, s3:PutObject on arn:aws:s3:::nexride-ml-artifacts/*
```

### Frontend on Vercel
```bash
# 1. Push to GitHub
# 2. Import project at vercel.com
# 3. Set root directory to: frontend/
# 4. No build command needed (static HTML)
# 5. Set env variable: VITE_API_URL=http://YOUR_EC2_IP
```

Or AWS Amplify:
```bash
# amplify.yml
version: 1
frontend:
  phases:
    build:
      commands:
        - echo "No build needed"
  artifacts:
    baseDirectory: frontend
    files:
      - '**/*'
```

## 🔌 API Reference

### POST /predict-demand
```json
// Request
{
  "hour": 8,
  "day_of_week": 0,
  "month": 6,
  "location": "Airport",
  "weather": "Rainy",
  "is_holiday": 0,
  "driver_availability": 45
}

// Response
{
  "demand_prediction": 149,
  "confidence_score": 0.912,
  "is_peak_hour": true,
  "location": "Airport",
  "latency_ms": 22.4
}
```

### POST /predict-price
```json
// Response
{
  "surge_multiplier": 2.35,
  "base_price": 25.0,
  "final_price": 58.75,
  "pricing_tier": "Very High",
  "latency_ms": 18.7
}
```

### POST /predict-all
Returns both demand + price in a single request.

### GET /options
Returns valid locations, weather conditions, hours.

### GET /metrics
Returns training metrics (RMSE, MAE, R²) for both models.

## 🧪 Testing

```bash
# Test API directly with curl
curl -X POST http://localhost:8000/predict-all \
  -H "Content-Type: application/json" \
  -d '{"hour":8,"day_of_week":0,"month":6,"location":"Airport","weather":"Rainy","driver_availability":30}'

# Run feature engineering smoke test
python src/feature_engineering.py

# Run prediction smoke test (models must exist)
python src/predict.py
```

## 📊 Power BI Dashboard

The simulator writes to `dashboard/dashboard_data.csv` with fields:
- `timestamp`, `location`, `weather`, `hour`
- `demand`, `surge`, `final_price_inr`
- `pricing_tier`, `is_peak_hour`, `total_latency_ms`

Import this CSV into Power BI for live analytics.

## 📤 GitHub Push Commands

```bash
# Initialize repository
git init
git add .
git commit -m "feat: NexRide AI — Uber Michelangelo-inspired ML platform

- Random Forest demand model (R²=0.986, 200 trees)
- Gradient Boosting surge model (R²=0.975, 200 estimators)  
- FastAPI with /predict-demand /predict-price /predict-all
- Animated frontend SPA with Chart.js dashboard
- Real-time simulator (sends requests every 2s)
- Full Docker + docker-compose setup
- AWS EC2 + S3 deployment guide"

# Push to GitHub
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/ride-ml-system.git
git push -u origin main
```

## 📸 Screenshots
<!-- Add screenshots here after deployment -->
- `docs/screenshots/home.png` — Hero page with particle animation
- `docs/screenshots/predict.png` — Live prediction results
- `docs/screenshots/dashboard.png` — Charts + heatmap
- `docs/screenshots/api-docs.png` — FastAPI Swagger UI

## 🧠 Tech Stack

| Layer | Technology |
|-------|------------|
| ML | scikit-learn (RandomForest, GradientBoosting) |
| API | FastAPI + Pydantic + Uvicorn |
| Frontend | HTML5 + Chart.js + Canvas API |
| Containerization | Docker + docker-compose |
| Reverse Proxy | Nginx |
| Cloud | AWS EC2 + S3 |
| Data | Pandas + NumPy (87,600 synthetic rows) |

## 📄 License
MIT
