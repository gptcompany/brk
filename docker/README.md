# Docker Setup for BRK

## Prerequisites

- Docker Engine (with buildx support)
- Docker Compose v2
- A running Bitcoin Core node with RPC enabled

## Quick Start

1. **Create environment file**
   ```bash
   cp docker/.env.example docker/.env
   ```
   Edit `docker/.env` and set `BITCOIN_DATA_DIR`, `BRK_DATA_DIR`, and RPC credentials for your node.

2. **Run with Docker Compose**
   ```bash
   docker compose -f docker/docker-compose.yml up -d
   ```

3. **Access BRK**
   - Web interface: http://localhost:7070
   - API: http://localhost:7070/api
   - Health check: http://localhost:7070/health

## Configuration

All configuration is passed via CLI args in `docker-compose.yml`. Edit the `command:` section to change settings.

### Environment Variables

These variables are interpolated into `docker-compose.yml` at startup:

| Variable | Description | Default |
|----------|-------------|---------|
| `BITCOIN_DATA_DIR` | Path to Bitcoin Core data directory | - |
| `BRK_DATA_DIR` | Host path for BRK data bind mount | `/media/sam/1TB/brk-data` |
| `BTC_RPC_HOST` | Bitcoin Core RPC host | `172.29.0.1` |
| `BTC_RPC_USER` | Bitcoin RPC username | `bitcoinrpc` |
| `BTC_RPC_PASSWORD` | Bitcoin RPC password | - |

### Connecting to Bitcoin Core

**Cookie File Authentication (Recommended)**
Uncomment the `--rpccookiefile` lines in `docker-compose.yml` and remove `--rpcuser`/`--rpcpassword`.

**Username/Password**
Set `BTC_RPC_USER` and `BTC_RPC_PASSWORD` in your `docker/.env` file.

**Network Connectivity**
- **Same host (Bitcoin Core running natively)**: use the bridge gateway declared in compose (`172.29.0.1`)
- **Same host (Bitcoin Core in Docker)**: Use the service name or container IP
- **Remote host**: Use the actual IP address or hostname

## Building

```bash
docker compose -f docker/docker-compose.yml build
```

## Data Storage

BRK uses [sparse files](https://en.wikipedia.org/wiki/Sparse_file). Volume inspection and `docker system df` may report the logical file size (>1 TB) instead of actual disk usage (~350 GB). Use `du -sh` on the volume mount point to see real usage.

### Bind Mount (Default)
Set `BRK_DATA_DIR` in `docker/.env` and make sure the directory exists and is writable by UID `1000`.

## Monitoring

```bash
docker compose -f docker/docker-compose.yml ps
docker compose -f docker/docker-compose.yml logs -f
docker inspect --format='{{.State.Health.Status}}' brk
```

## Troubleshooting

### Cannot connect to Bitcoin Core
1. Ensure Bitcoin Core is running with `-server=1`
2. Check RPC credentials are correct
3. Verify the container can reach `172.29.0.1:8332`
4. If Bitcoin Core only binds RPC on `127.0.0.1`, add `rpcbind=0.0.0.0` and allow `rpcallowip=172.29.0.0/24`

### Permission denied errors
Ensure the Bitcoin data directory is readable by the container user (UID 1000).

## Security

- Bitcoin data is mounted read-only
- BRK runs as non-root user inside container
- BRK only binds to `127.0.0.1:7070` on the host
