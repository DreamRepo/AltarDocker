# AltarDocker — Deployment Guide

Complete guide for deploying MongoDB and MinIO infrastructure for Sacred experiment tracking.

**Services included:**
- MongoDB for experiment metadata
- MinIO for S3-compatible object storage

> **Note:** Omniboard is managed by [AltarViewer](../AltarViewer), AltarExtractor is [deployed separately](../AltarExtractor).

---

## Prerequisites
- Docker >= 24 and Docker Compose v2 installed
- At least 2 GB free disk space

---

## 1) (Optional) Customize Settings

**The compose file works without a `.env` file!** It has built-in defaults:
- MongoDB: `sacred` database on port `27017`
- MinIO: `minio_admin` / `changeme123` on ports `9000` and `9001`
- Data stored in `./data/mongo/` and `./data/minio/`

**To customize**, copy `.env.example` to `.env` and modify the values:

```bash
cp .env.example .env
# Edit .env with your preferred values
```

The `.env.example` file contains:

```dotenv
# MongoDB Configuration
MONGO_DB=sacred
MONGO_PORT=27017

# MinIO Configuration
MINIO_ROOT_USER=minio_admin
MINIO_ROOT_PASSWORD=your_secure_password
MINIO_S3_PORT=9000
MINIO_CONSOLE_PORT=9001

# Volume Paths (where data is stored)
MONGO_DATA_PATH=./data/mongo/
MINIO_DATA_PATH=./data/minio/
```

---

## 2) Start the Services

```bash
docker compose up -d
docker ps
```

> **Note:** On Linux, you may need to prefix commands with `sudo`.

**All services start automatically:**
- MongoDB on port 27017
- MinIO S3 API on port 9000
- MinIO Console on port 9001

### Access URLs

| Service        | URL                                                      |
|----------------|----------------------------------------------------------|
| MongoDB        | `mongodb://localhost:27017`                              |
| MinIO S3 API   | http://localhost:9000                                    |
| MinIO Console  | http://localhost:9001                                    |

---


## Backups (Linux)

### MongoDB backup script

Edit the `mongo_dump.sh` file:
```bash
#!/bin/bash

# Dumps root repository
BAS5_DIR="/home/user/mongodumps/dump"  # Change to your backup location

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

---6

## 7) Cleanup

```bash
docker compose down
# To also remove volumes (deletes data):
# docker compose down -v
```

---

## Troubleshooting

- **Omniboard cannot connect?** Use [AltarViewer](../AltarViewer) to launch Omniboard properly configured for your database.
- **Port conflict?** Change `MONGO_PORT` or MinIO port mappings in `docker-compose.yml`.
- **S3 SDKs:** Use endpoint `http://localhost:9000` with MinIO credentials.

---

## Related

- [AltarExtractor](https://github.com/DreamRepo/AltarExtractor) — Browse and filter Sacred experiments (standalone deployment)
- [AltarSender](https://github.com/DreamRepo/AltarSender) — GUI to send experiments to Sacred and MinIO
- [AltarViewer](https://github.com/DreamRepo/AltarViewer) — Launch Omniboard configured for your database
