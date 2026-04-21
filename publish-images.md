# Publish And Share Docker Images (No Source)

This guide lets others run Entrix from container images only, without access to your source repository.

## What This Includes

- Build and push backend/frontend images from your private repo.
- Share runtime files only (`docker-compose.runtime.yml` and `.env`).
- Consumers run with Docker Compose and never need your source code.

## 1. Build And Push Images (Owner Side)

Run from the repo root:

```bash
docker login
docker build -t oussama2200/entrix-backend:v1 ./rfid-backend
docker build -t oussama2200/entrix-frontend:v1 ./rfid-frontend
docker push oussama2200/entrix-backend:v1
docker push oussama2200/entrix-frontend:v1
```

Use a new image tag (`v2`, `v3`, etc.) whenever you release updates.

## 2. Prepare Runtime Bundle (What You Share)

Share only these files:

- `docker-compose.runtime.yml`
- `.env.runtime.example`

The consumer should create `.env` from the example and fill image names/tags.

## 3. Run From Images Only (Consumer Side)

PowerShell:

```powershell
Copy-Item .env.runtime.example .env
# edit .env and replace image placeholders

docker compose --env-file .env -f docker-compose.runtime.yml pull
docker compose --env-file .env -f docker-compose.runtime.yml up -d
```

Unix/macOS shell:

```bash
cp .env.runtime.example .env
# edit .env and replace image placeholders

docker compose --env-file .env -f docker-compose.runtime.yml pull
docker compose --env-file .env -f docker-compose.runtime.yml up -d
```

## 4. Verify

- App: `http://localhost`
- API health: `http://localhost:3001/api/health`

## 5. Stop

```bash
docker compose --env-file .env -f docker-compose.runtime.yml down
```

## Security Note

This approach keeps your Git repository and commit history private, but it is not full code secrecy. JavaScript bundles and runtime artifacts can still be inspected.
