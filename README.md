# AltarDocker

Docker Compose stack for running Sacred ML experiment tracking infrastructure locally.

## What's Included

| Service | Description | Port |
|---------|-------------|------|
| **MongoDB** | Stores experiment metadata | 27077 |
| **Omniboard** | Web dashboard for Sacred experiments | 9030 |
| **MinIO** | S3-compatible object storage (optional) | 9000, 9001 |
| **AltarExtractor** | Browse and filter experiments (optional) | 8050 |

## Quick Start

1. **Create `.env` file** with your credentials:
   ```dotenv
   MONGO_DB=sacred
   MINIO_ROOT_USER=minio_admin
   MINIO_ROOT_PASSWORD=your_minio_password
   OMNIBOARD_HOST_PORT=9030
   EXTRACTOR_HOST_PORT=8050
   ```

2. **Start the stack:**
   ```bash
   # Basic (MongoDB + Omniboard)
   docker compose up -d

   # With MinIO
   docker compose --profile minio up -d

   # With AltarExtractor
   docker compose --profile extractor up -d

   # Full stack
   docker compose --profile minio --profile extractor up -d
   ```

3. **Access the services:**
   - Omniboard: http://localhost:9030
   - AltarExtractor: http://localhost:8050
   - MinIO Console: http://localhost:9001

## Documentation

- [DEPLOY.md](DEPLOY.md) — Full deployment guide with detailed configuration
- [MANAGE_USERS.md](MANAGE_USERS.md) — MongoDB and MinIO user management


## For developers: 

### Purpose: add credentials to MongoDB
Update the .env file: 

   ```dotenv
   MONGO_ROOT_USER=admin
   MONGO_ROOT_PASSWORD=your_secure_password
   ```

and the "environment" section of docker-compose.yml file:

 ```docker
  mongo:
    image: mongo:6
    container_name: mongo
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_ROOT_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
      MONGO_INITDB_DATABASE: ${MONGO_DB}
    ports:
      - "27077:27017"
    volumes:
      - PATH/TO/DATA/mongo_data:/data/db

 ```

## Requirements

- Docker >= 24
- Docker Compose v2
- At least 2 GB free disk space

## Related

- [AltarExtractor](https://github.com/DreamRepo/AltarExtractor) — Browse and filter Sacred experiments in a web UI
- [AltarSender](https://github.com/DreamRepo/AltarSender) — GUI to send experiments to Sacred and MinIO

## License

GNU General Public License v3.0

