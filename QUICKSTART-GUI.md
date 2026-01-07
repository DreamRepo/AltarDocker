# AltarDocker - Quick Start for Docker Desktop Users

**No coding or terminal required!** This guide is for users who prefer Docker Desktop's graphical interface.

## Prerequisites

1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Make sure Docker Desktop is running

## Simple Setup (3 Steps)

### 1. Download the File

Download `docker-compose_default.yml` from the [AltarDocker repository](https://github.com/DreamRepo/AltarDocker) to a folder on your computer.
Copy the full path of this folder. 

![image](https://github.com/DreamRepo/AltarDocker/compose_path.png)


### 2. Deploy with Docker Desktop

**Using Docker Desktop GUI - if you find the option**
1. Open Docker Desktop
2. Click the **"+"** button or **"Import"** in the Containers tab
3. Select your `docker-compose_default.yml` file
4. Click **"Run"**

**Using Terminal:**
```bash
docker compose -f path/to/your/folder/docker-compose_default.yml up -d
```
![video](https://github.com/DreamRepo/AltarDocker/install_altardocker.mp4)

### 3. Access Your Services

- **MinIO Console**: http://localhost:9001
  - Username: `minio_admin`, Password: `changeme123`
- **MongoDB**: `mongodb://localhost:27017`

That's it! Your services are running with sensible defaults.

## Want to Customize?

**Simple method** - Edit the compose file directly:
- Open `docker-compose_default.yml` in Notepad/VS Code
- Find lines with `:-` (e.g., `:-changeme123`)
- Change the value after `:-` to customize

**Advanced method** - See [DEPLOY.md](DEPLOY.md) for:
- Using `.env` files
- Changing ports
- Custom data storage paths
- Authentication setup

## Managing Services

In Docker Desktop:
- **View status**: Go to "Containers" tab → see `mongo_altar` (and `minio_altar`)
- **Stop**: Click the Stop button
- **Restart**: Click the Start button
- **View logs**: Click the container name

## Troubleshooting

**Port conflict?**
- Edit compose file and change `:-27017` to `:-27077` (see customization above)

**Can't access services?**
- Check containers are "Running" in Docker Desktop
- See [DEPLOY.md](DEPLOY.md) for detailed troubleshooting

## Next Steps

- Download [AltarSender](https://github.com/DreamRepo/AltarSender/releases) to upload experiments
- Download [AltarViewer](https://github.com/DreamRepo/AltarViewer/releases) to visualize your data
- Visit [AltarExtractor](https://github.com/DreamRepo/AltarExtractor) to analyze experiments
