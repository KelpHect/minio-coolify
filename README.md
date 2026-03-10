# MinIO Docker Images

Automated builds of [MinIO](https://github.com/minio/minio) Docker images from official releases.

Since MinIO stopped providing pre-built Docker images for new releases, this repository automatically builds and publishes them to both GitHub Container Registry and Docker Hub.

> If you need a specific version / release, open an issue. I (Andras) will make it available.

## Available Images

Images are available at:

**GitHub Container Registry (GHCR):**
```
ghcr.io/coollabsio/minio:<tag>
```

**Docker Hub:**
```
docker.io/coollabsio/minio:<tag>
# or simply
coollabsio/minio:<tag>
```

### Tags

- `latest` - Latest MinIO release
- `RELEASE.2024-10-13T13-34-11Z` - Specific release version (full release name)
- `2024-10-13T13-34-11Z` - Specific release version (without RELEASE. prefix)

## Usage

### Basic Usage

```bash
# Using Docker Hub (simpler, no authentication needed)
docker run -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=minioadmin \
  -v /path/to/data:/data \
  coollabsio/minio:latest \
  server /data --console-address ":9001"

# Or using GitHub Container Registry
docker run -p 9000:9000 -p 9001:9001 \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=minioadmin \
  -v /path/to/data:/data \
  ghcr.io/coollabsio/minio:latest \
  server /data --console-address ":9001"
```

### Docker Compose (1-click deployment)

A ready-to-use `docker-compose.yml` is included in this repository.

```bash
# 1. Copy the example environment file and edit credentials
cp .env.example .env

# 2. Start MinIO
docker compose up -d
```

- **S3 API**: http://localhost:9000
- **Web Console**: http://localhost:9001

All settings (credentials, ports) are configured via the `.env` file. See `.env.example` for available options.

<details>
<summary>Compose file contents</summary>

```yaml
services:
  minio:
    image: coollabsio/minio:latest
    container_name: minio
    restart: unless-stopped
    ports:
      - "${MINIO_API_PORT:-9000}:9000"
      - "${MINIO_CONSOLE_PORT:-9001}:9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    volumes:
      - minio-data:/data
    command: server /data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
      start_period: 10s

volumes:
  minio-data:
```

</details>

## Building Locally

To build an image locally:

```bash
# Build latest version
docker build -t minio:test .

# Build specific version
docker build --build-arg MINIO_VERSION=RELEASE.2024-10-13T13-34-11Z -t minio:test .
```

## Architecture

- **Multi-platform**: Built for both `linux/amd64` and `linux/arm64`
- **Source-based**: Compiled from official MinIO source code at each release
- **Minimal base**: Uses Red Hat UBI9 micro image for security and size
- **Official scripts**: Uses MinIO's official docker-entrypoint.sh and other scripts from the cloned repository

## Ports

- **9000**: MinIO S3 API endpoint
- **9001**: MinIO Web Console (admin UI)

## Environment Variables

- `MINIO_ROOT_USER`: Root access key (username)
- `MINIO_ROOT_PASSWORD`: Root secret key (password)
- `MINIO_CONFIG_ENV_FILE`: Path to configuration file
- See [MinIO documentation](https://min.io/docs/minio/linux/reference/minio-server/minio-server.html) for more options

## License

This build automation is provided as-is. MinIO itself is licensed under AGPL-3.0.

**Note**: According to MinIO's license terms, production use of compiled-from-source binaries is at your own risk. For production deployments, MinIO recommends their enterprise offerings.

## Links

- [MinIO Official Repository](https://github.com/minio/minio)
- [MinIO Documentation](https://min.io/docs/minio/)
- [MinIO Releases](https://github.com/minio/minio/releases)
