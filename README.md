# Fluhoms Runner

Self-hosted ETL execution agent. Deploy the Fluhoms Runner on Docker, Kubernetes, or any container orchestrator to run your data pipelines securely within your infrastructure.

**Multi-architecture**: `linux/amd64` and `linux/arm64` supported.

## Available Tags

| Tag | Description |
|-----|-------------|
| `latest` | Latest stable release |
| `beta` | Latest beta release |
| `x.y.z` | Specific version (e.g. `1.2.0`) |

## Quick Start

```bash
docker run -d \
  --name fluhoms-runner \
  -e ACCEPT_LICENSE=yes \
  -e REGISTER_CODE=your-register-code \
  -v fluhoms-runner-data:/data \
  ghcr.io/fluhoms/fluhoms-runner:latest
```

> **License acceptance**: `ACCEPT_LICENSE=yes` is required on first launch. Once accepted, the flag is persisted in the `/data` volume.

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `ACCEPT_LICENSE` | Accept the license agreement (`yes` required on first run) | - |
| `REGISTER_CODE` | Registration code for auto-registration (see below) | - |
| `LOG_TO_FILE` | Enable file logging with log rotation | `false` |
| `FLUHOMS_DATA_PATH` | Data directory path | `/data` |
| `FLUHOMS_LOG_PATH` | Logs directory path (when `LOG_TO_FILE=true`) | `/data/logs` |

### Registration

The `REGISTER_CODE` is generated from your Fluhoms workspace. When provided, the runner **automatically registers itself** on startup if it is not already registered.

- If the runner is already registered with the same code, nothing happens
- If the code changes, the runner re-registers with the new code
- The registration state is persisted in the `/data` volume

You can also register manually without using the environment variable:

```bash
docker exec -it fluhoms-runner fluhoms_runner register YOUR_CODE
```

## Data Persistence

Mount a volume to `/data` to persist runner data across container restarts:

```bash
docker run -d \
  --name fluhoms-runner \
  -e ACCEPT_LICENSE=yes \
  -v fluhoms-runner-data:/data \
  ghcr.io/fluhoms/fluhoms-runner:latest
```

You can also use an env file:

```bash
docker run -d \
  --name fluhoms-runner \
  --env-file .env \
  -v fluhoms-runner-data:/data \
  ghcr.io/fluhoms/fluhoms-runner:latest
```

## Docker Compose

```yaml
services:
  fluhoms-runner:
    image: ghcr.io/fluhoms/fluhoms-runner:latest
    container_name: fluhoms-runner
    restart: unless-stopped
    environment:
      - ACCEPT_LICENSE=yes
      - TZ=Europe/Paris
    volumes:
      - fluhoms-runner-data:/data
    logging:
      driver: json-file
      options:
        max-size: "50m"
        max-file: "5"

volumes:
  fluhoms-runner-data:
```

```bash
docker compose up -d
```

## Kubernetes

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fluhoms-runner-data
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fluhoms-runner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fluhoms-runner
  template:
    metadata:
      labels:
        app: fluhoms-runner
    spec:
      containers:
        - name: fluhoms-runner
          image: ghcr.io/fluhoms/fluhoms-runner:latest
          env:
            - name: ACCEPT_LICENSE
              value: "yes"
            - name: REGISTER_CODE
              valueFrom:
                secretKeyRef:
                  name: fluhoms-runner-secret
                  key: register-code
          volumeMounts:
            - name: data
              mountPath: /data
          livenessProbe:
            httpGet:
              path: /status
              port: 2024
            initialDelaySeconds: 15
            periodSeconds: 30
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: fluhoms-runner-data
```

## Health Check

The image includes a built-in health check. Verify the container status:

```bash
docker inspect --format='{{.State.Health.Status}}' fluhoms-runner
```

## Logging

By default, logs are sent to **stdout/stderr only**, which integrates with Docker and Kubernetes logging drivers.

To enable **file logging** with automatic rotation, set `LOG_TO_FILE=true`:

```bash
docker run -d \
  --name fluhoms-runner \
  -e ACCEPT_LICENSE=yes \
  -e LOG_TO_FILE=true \
  -v fluhoms-runner-data:/data \
  ghcr.io/fluhoms/fluhoms-runner:latest
```

When enabled, log rotation runs automatically:

- **Frequency**: Daily or when a file exceeds 100 MB
- **Retention**: 7 compressed files, up to 30 days

## Useful Commands

```bash
# View logs
docker logs -f fluhoms-runner

# Shell into the container
docker exec -it fluhoms-runner /bin/bash

# Graceful shutdown
docker stop fluhoms-runner

# Restart
docker restart fluhoms-runner
```

## Links

- 🌐 [Website](https://fluhoms.io)
- 📖 [Documentation](https://docs.fluhoms.io)
- 💬 [Support](mailto:support@fluhoms.io)
