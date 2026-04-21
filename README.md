# Entrix RFID Access Control

Entrix is a full-stack RFID access control platform with a simulation console, live scan stream, rule-based authorization, and PostgreSQL persistence.

## Table of Contents

- [What this project includes](#what-this-project-includes)
- [Screenshots](#screenshots)
- [Repository structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Quick start with Docker](#quick-start-with-docker)
- [Share as Docker Images (no source)](#share-as-docker-images-no-source)
- [Local development without Docker](#local-development-without-docker)
- [Environment variables](#environment-variables)
- [API overview](#api-overview)
- [Realtime events](#realtime-events)
- [Captor (reader) setup guide](#captor-reader-setup-guide)
- [Troubleshooting](#troubleshooting)

## What this project includes

- Backend API using Express + TypeScript.
- PostgreSQL schema creation on startup.
- Frontend dashboard using React + Vite + TypeScript.
- Realtime updates with Socket.IO.
- Simulation endpoint to trigger card scans.
- Seed/reset flows for demo data.

## Screenshots

| Simulator                                            | Dashboard                                            |
| ---------------------------------------------------- | ---------------------------------------------------- |
| ![Entrix screenshot 1](docs/screenshots/entrix1.png) | ![Entrix screenshot 2](docs/screenshots/entrix2.png) |

| Cards                                                | Readers                                              |
| ---------------------------------------------------- | ---------------------------------------------------- |
| ![Entrix screenshot 3](docs/screenshots/entrix3.png) | ![Entrix screenshot 4](docs/screenshots/entrix4.png) |

![Entrix screenshot 5](docs/screenshots/entrix5.png)

## Repository structure

```text
.
|- docker-compose.yml
|- rfid-backend/
|  |- src/
|  |  |- adapters/inbound/rest/      # REST routes
|  |  |- adapters/inbound/websocket/ # Socket handler registration
|  |  |- domain/                     # Models + service logic
|  |  |- infrastructure/             # DB, DI, seed, realtime emitter
|- rfid-frontend/
|  |- src/
|  |  |- pages/                      # Dashboard, cards, readers, events, rules, simulator
|  |  |- lib/                        # API and socket clients
```

## Prerequisites

- Docker and Docker Compose, or
- Node.js 20+ and PostgreSQL 16+ for local execution.

## Quick start with Docker

1. From the repository root, build and start services:

```bash
docker compose up --build
```

2. Open:

- App: http://localhost
- Backend health: http://localhost:3001/api/health
- Adminer: http://localhost:8080

3. In the app, open **Scanner** and click **Seed DB** to load demo cards, readers, rules, and scan history.

### Important credential alignment note

The backend container uses `DATABASE_URL` from `docker-compose.yml`. Ensure PostgreSQL service credentials match that URL.

Current compose values should be aligned in the same file:

- `POSTGRES_USER`
- `POSTGRES_PASSWORD`
- `DATABASE_URL`
- Postgres healthcheck user in `pg_isready`

If these values differ, backend startup and health checks can fail.

## Share as Docker Images (no source)

Use this project as a private source repo and distribute only Docker images + runtime files.

- Full step-by-step guide: [docs/publish-images.md](docs/publish-images.md)
- Runtime compose: `docker-compose.runtime.yml`
- Runtime env template: `.env.runtime.example`

Quick run from published images:

```powershell
Copy-Item .env.runtime.example .env
# edit .env and set your image names/tags
docker compose --env-file .env -f docker-compose.runtime.yml pull
docker compose --env-file .env -f docker-compose.runtime.yml up -d
```

## Local development without Docker

### 1) Backend

```bash
cd rfid-backend
npm install
npm run dev
```

Backend runs on `http://localhost:3001` by default.

### 2) Frontend

```bash
cd rfid-frontend
npm install
npm run dev
```

Frontend runs on `http://localhost:5173` by default.

## Environment variables

### Backend (`rfid-backend`)

- `PORT`: API port (default `3001`)
- `DATABASE_URL`: Postgres connection string
- `CORS_ORIGIN`: allowed frontend origin (default `http://localhost:5173`)

Example:

```env
PORT=3001
DATABASE_URL=postgresql://rfid_admin:secure_password_123@localhost:5432/rfid_system
CORS_ORIGIN=http://localhost:5173
```

### Frontend (`rfid-frontend`)

- `VITE_API_BASE_URL`: REST base URL
  - dev default: `http://localhost:3001/api`
  - prod default: `/api`
- `VITE_SOCKET_URL`: Socket.IO base URL
  - dev default: `http://localhost:3001`
  - prod default: current browser origin

Example:

```env
VITE_API_BASE_URL=http://localhost:3001/api
VITE_SOCKET_URL=http://localhost:3001
```

## API overview

### Health

- `GET /api/health`

### Cards

- `GET /api/cards`
- `GET /api/cards/:id`
- `POST /api/cards`
- `PUT /api/cards/:id`
- `DELETE /api/cards/:id`

### Readers (captors)

- `GET /api/readers`
- `GET /api/readers/:id`
- `POST /api/readers`
- `PUT /api/readers/:id`
- `DELETE /api/readers/:id`

### Scan events

- `GET /api/scan-events`
- `GET /api/scan-events/stats/summary`

### Access rules

- `GET /api/access-rules`
- `GET /api/access-rules/:id`
- `POST /api/access-rules`
- `PUT /api/access-rules/:id`
- `DELETE /api/access-rules/:id`

### Simulation

- `POST /api/simulation/scan`
- `POST /api/simulation/seed`
- `DELETE /api/simulation/reset`

## Realtime events

The frontend subscribes to Socket.IO event:

- `scan:realtime`

This event powers the live activity feed in the scanner page.

## Captor (reader) setup guide

Detailed guide:

- [docs/captors-configuration.md](docs/captors-configuration.md)

It explains:

- How to add captors/readers
- How `doorId` and `direction` must be paired
- How to configure frontend/backend env for captor usage
- How to validate and troubleshoot captor behavior

## Troubleshooting

### Live activity feed stays empty

- Verify frontend can connect to Socket.IO (sidebar status should be Connected).
- Verify backend emits `scan:realtime` on scan simulation.
- Ensure frontend `VITE_SOCKET_URL` points to backend socket origin.

### Scanner says no reader configured for selected door

- Make sure at least one reader exists with that `doorId` and `direction`.
- For full door simulation, create both ENTRY and EXIT readers sharing the same `doorId`.

### API calls fail due to CORS

- Set backend `CORS_ORIGIN` to your frontend origin.

### DB connection errors on startup

- Check `DATABASE_URL` and PostgreSQL credentials.
- Ensure database host/port are reachable.
