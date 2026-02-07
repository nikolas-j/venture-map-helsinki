# Venture Map Helsinki

This README describes the private reposiory for the following web app: Interactive map-based directory visualizing Helsinki's startup ecosystem. Browse companies, filter by industry and funding stage, and submit your own startup.

**Live Application**: https://venture-map-helsinki.vercel.app

![web app screenshot](docs/venture-map-ss.png)

## Overview

This project provides a public-facing platform for exploring Helsinki-based startups on an interactive Mapbox-powered map. Users can filter companies by industry tags, funding stage, company size, and hiring status. Founders can submit their startups via a public form and edit entries using a secret key.

## Tech Stack

### Frontend
- **Framework**: React 18, TypeScript, Vite
- **Map**: Mapbox GL JS
- **Styling**: Tailwind CSS with Google Fonts
- **Deployment**: Vercel (automatic deployment on push to master)

### Backend
- **Framework**: FastAPI, SQLAlchemy 2.0, Pydantic v2
- **Database**: PostgreSQL (Supabase)
- **Storage**: Supabase Storage (logo images)
- **Email**: Gmail SMTP for feedback delivery
- **Deployment**: Docker + AWS ECR + App Runner (automated via GitHub Actions)

## Architecture

### API Communication
The frontend fetches company data once on mount and performs all filtering client-side for instant updates. API endpoints provide:

- **GET /api/v1/companies** - List all companies (filtered server-side by query params)
- **GET /api/v1/companies/{id}** - Company details
- **GET /api/v1/metrics** - Ecosystem aggregate statistics
- **POST /api/v1/submissions** - Submit startup for review
- **PATCH /api/v1/submissions/{secret_key}** - Edit submission
- **POST /api/v1/media/submission-logo/{id}/{secret_key}** - Upload logo
- **POST /api/v1/feedback** - Send feedback via email

### Caching & Rate Limiting

**Caching**:
- Ecosystem metrics endpoint cached for 5 minutes (backend)
- Companies list fetched once on frontend mount (no client-side cache)

**Rate Limiting** (Redis-backed, per endpoint):
- Company queries: 60 requests/minute
- Submissions: 4 requests/minute
- Logo uploads: 3 requests/minute
- Feedback: 3 requests/minute

### Submission Workflow

1. User submits startup via public form (reCAPTCHA v2 protected)
2. Receives secret key for future edits
3. Can upload logo (PNG auto-cropped to 128x128, SVG supported)
4. Submission enters review queue
5. After approval, appears on production map

## Deployment

### Frontend (Vercel)
- Root directory: `frontend/`
- Automatic deployment on push to `master` branch
- Environment variables configured in Vercel dashboard

### Backend (AWS)
- Docker image built via GitHub Actions workflow
- Pushed to AWS ECR on push to `master` branch
- AWS App Runner pulls `:latest` tag and redeploys automatically

## Requirements

### Development
- Node.js 18+
- Python 3.11+ with uv package manager
- Docker & Docker Compose
- PostgreSQL (or Supabase account)

### API Keys
- Mapbox access token
- Google reCAPTCHA v2 keys (site + secret)
- Supabase credentials (database URL, project URL, secret key)
- Gmail SMTP credentials (for feedback emails)
- AWS credentials (for deployment: ECR access)

## Local Setup

```bash
# Frontend
cd frontend
npm install
cp .env.example .env  # Add VITE_API_BASE_URL, VITE_MAPBOX_TOKEN, VITE_RECAPTCHA_SITE_KEY
npm run dev           # Runs on http://localhost:5173

# Backend
cd backend
uv sync
cp .env.example app/.env  # Add database, SMTP, reCAPTCHA credentials
docker-compose up -d      # Or: uv run uvicorn app.main:app --reload
```

## Documentation

- **Frontend**: [frontend/README.md](frontend/README.md) - Component architecture, styling guide, Vercel deployment
- **Backend**: [backend/README.md](backend/README.md) - API endpoints, schemas, services, configuration
- **AWS Deployment**: [docs/AWS_FLOW.md](docs/AWS_FLOW.md) - ECR setup, Docker build/push commands

## Project Structure

```
.
├── frontend/          # React + TypeScript + Mapbox frontend
├── backend/           # FastAPI + SQLAlchemy backend
└── .github/
    └── workflows/     # GitHub Actions (automated ECR deployment)
```
