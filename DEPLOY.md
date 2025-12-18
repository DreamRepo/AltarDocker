# AltarDocker — Local Deployment Guide

This guide helps you start a minimal stack locally using Docker Compose:
- MongoDB for experiment metadata
- MinIO (S3-compatible) for raw data (optional)
- Omniboard connected to MongoDB
- AltarExtractor for browsing Sacred experiments (optional)

---

## Prerequisites
- Docker >= 24 and Docker Compose v2 installed (https://docs.docker.com/engine/install/)
- At least 2 GB of free disk space (more depending on the size of your data)

---

## Project Layout
```
AltarDocker/
├─ .env                  # Environment variables (create this)
├─ docker-compose.yml    # Docker Compose configuration
├─ DEPLOY.md             # This file
├─ MANAGE_USERS.md       # User management guide
└─ mongo_dump.sh         # Backup script example
```

---

## 1) Create the `.env` file

Create a `.env` file in this folder with strong passwords:

```dotenv
# MongoDB (root account for initial setup)
MONGO_ROOT_USER=admin
MONGO_ROOT_PASSWORD=change_me_mongo_password
MONGO_DB=sacred

# MinIO (S3 + console)
MINIO_ROOT_USER=minio_admin
MINIO_ROOT_PASSWORD=change_me_minio_password

# Host port for Omniboard (internal port is 9000)
OMNIBOARD_HOST_PORT=9004

# Host port for AltarExtractor (internal port is 8050)
EXTRACTOR_HOST_PORT=8050
```

---

## 2) Edit `docker-compose.yml`

The data for MinIO and MongoDB will be saved in the folders specified in `volumes` (outside the Docker container). Update the volume paths to your preferred locations:

```yaml
services:
  mongo:
    image: mongo:6
    container_name: mongo
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_ROOT_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
      MONGO_INITDB_DATABASE: ${MONGO_DB}
    ports:
      - "27017:27017"
    volumes:
      - /path/to/your/mongo_data:/data/db  # Change this path

  minio:
    image: quay.io/minio/minio:latest
    container_name: minio
    restart: unless-stopped
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    ports:
      - "9000:9000"   # S3 API
      - "9001:9001"   # Web console
    volumes:
      - /path/to/your/minio_data:/data  # Change this path
    profiles:
      - minio

  omniboard:
    image: vivekratnavel/omniboard:latest
    container_name: omniboard
    restart: unless-stopped
    depends_on:
      - mongo
    ports:
      - "${OMNIBOARD_HOST_PORT}:9000"
    command: [
      "--mu",
      "mongodb://${MONGO_ROOT_USER}:${MONGO_ROOT_PASSWORD}@mongo:27017/?authSource=admin&authMechanism=SCRAM-SHA-1",
      "${MONGO_DB}"
    ]

  extractor:
    build:
      context: ../AltarExtractor
      dockerfile: Dockerfile
    container_name: altar-extractor
    restart: unless-stopped
    depends_on:
      - mongo
    ports:
      - "${EXTRACTOR_HOST_PORT:-8050}:8050"
    environment:
      - PORT=8050
      - SACRED_DB_NAME=${MONGO_DB}
    profiles:
      - extractor

volumes:
  mongo_data:
  minio_data:
```

---

## 3) Start the Docker containers

### Basic stack (MongoDB + Omniboard)
```bash
docker compose up -d
docker ps
```

### With MinIO
```bash
docker compose --profile minio up -d
docker ps
```

### With AltarExtractor
```bash
docker compose --profile extractor up -d
docker ps
```

### With both MinIO and AltarExtractor
```bash
docker compose --profile minio --profile extractor up -d
docker ps
```

> **Note:** On Linux, you may need to prefix commands with `sudo`.

### URLs

| Service        | URL                                                      |
|----------------|----------------------------------------------------------|
| MongoDB        | `mongodb://localhost:27017` (authenticate with root)     |
| MinIO S3 API   | http://localhost:9000                                    |
| MinIO Console  | http://localhost:9001                                    |
| Omniboard      | http://localhost:9004 (or your `OMNIBOARD_HOST_PORT`)    |
| AltarExtractor | http://localhost:8050 (or your `EXTRACTOR_HOST_PORT`)    |

---

## 4) Connecting AltarExtractor to MongoDB

When you open AltarExtractor in your browser, you need to configure the MongoDB connection:

1. Open http://localhost:8050 (or your configured `EXTRACTOR_HOST_PORT`)
2. In the "Database credentials" section, enter:
   - **Host**: `mongo` (the Docker service name, not localhost)
   - **Port**: `27017`
   - **Username**: your `MONGO_ROOT_USER` value
   - **Password**: your `MONGO_ROOT_PASSWORD` value
   - **Auth source**: `admin`
3. Enter your database name (e.g., `sacred`)
4. Click "Connect"

> **Tip:** Check "Save credentials" to remember your connection settings in browser local storage.

---

## 5) (Recommended) Create an app-specific MongoDB user

```bash
docker exec -it mongo mongosh -u admin -p your_password --authenticationDatabase admin
```

Inside mongosh:
```javascript
use sacred
db.createUser({
  user: "sacred_rw",
  pwd:  "change_me",
  roles: [ { role: "readWrite", db: "sacred" } ]
})
```

Then update Omniboard to use this user (edit `docker-compose.yml`):
```yaml
command: [
  "--mu",
  "mongodb://sacred_rw:change_me@mongo:27017/?authSource=sacred&authMechanism=SCRAM-SHA-1",
  "sacred"
]
```

Restart Omniboard:
```bash
docker compose up -d omniboard
```

> **Note:** On Linux, prefix with `sudo` if needed.

---

## 6) Backups (Linux)

### MongoDB backup script

Edit the `mongo_dump.sh` file:
```bash
#!/bin/bash

# Dumps root repository
BASE_DIR="/home/user/mongodumps/dump"  # Change to your backup location

# Timestamp (e.g., 2025-07-07)
DATE_STR=$(date +%Y-%m-%d)

# Destination folder
OUT_DIR="$BASE_DIR/$DATE_STR"

# Execute the dump
/usr/bin/mongodump --host 0.0.0.0 --port 27017 \
  --username "$MONGO_ROOT_USER" \
  --password "$MONGO_ROOT_PASSWORD" \
  --authenticationDatabase admin \
  --out="$OUT_DIR"

# Keep only the last 3 dumps
cd "$BASE_DIR"
ls -dt */ | tail -n +4 | xargs -d '\n' rm -rf
```

Make it executable:
```bash
chmod +x mongo_dump.sh
```

Schedule with crontab (runs daily at 2am):
```bash
crontab -e
```
Add:
```
0 2 * * * bash /path/to/mongo_dump.sh >> /path/to/cron.log 2>&1
```

---

## 7) Cleanup

```bash
docker compose down
# To also remove volumes (deletes data):
# docker compose down -v
```

---

## Troubleshooting

- **Omniboard cannot connect?** Check the `--mu` URI (authSource, host, port) and that MongoDB is reachable.
- **Port conflict?** Change `OMNIBOARD_HOST_PORT`, `EXTRACTOR_HOST_PORT`, or MinIO mappings in `docker-compose.yml`.
- **S3 SDKs:** Use endpoint `http://localhost:9000` with MinIO credentials.
- **AltarExtractor can't connect?** Make sure to use `mongo` as the host (Docker service name), not `localhost`.
