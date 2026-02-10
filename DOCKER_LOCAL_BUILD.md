# Building Activepieces from Local Source with Docker

This guide explains how to build and run Activepieces from your local source code using Docker Compose, including all required dependencies (PostgreSQL and Redis).

## Prerequisites

- Docker Engine (v20.10 or higher)
- Docker Compose v2
- At least 4GB of available RAM
- 10GB of free disk space

## Quick Start

### 1. Environment Configuration

The `.env` file has been generated with secure random passwords and encryption keys. You can review and modify it if needed:

```bash
cat .env
```

**Important environment variables:**
- `AP_FRONTEND_URL`: The URL where Activepieces will be accessible (default: http://localhost:8080)
- `AP_POSTGRES_PASSWORD`: PostgreSQL password (auto-generated)
- `AP_ENCRYPTION_KEY`: 256-bit encryption key (auto-generated)
- `AP_JWT_SECRET`: JWT signing secret (auto-generated)

### 2. Build and Start Services

Build the Docker image from local source and start all services:

```bash
docker compose -f docker-compose.local.yml up --build -d
```

This will:
- Build the Activepieces application from your local source code
- Start PostgreSQL database
- Start Redis cache
- Create persistent volumes for data storage

### 3. Access Activepieces

Once all services are running, access Activepieces at:
- **Web Interface**: http://localhost:8080

### 4. Monitor Logs

View logs from all services:
```bash
docker compose -f docker-compose.local.yml logs -f
```

View logs from specific service:
```bash
docker compose -f docker-compose.local.yml logs -f activepieces
docker compose -f docker-compose.local.yml logs -f postgres
docker compose -f docker-compose.local.yml logs -f redis
```

### 5. Check Service Health

Check the status of all services:
```bash
docker compose -f docker-compose.local.yml ps
```

## Production Deployment

For production deployments, ensure you:

1. **Update the frontend URL** in `.env`:
   ```
   AP_FRONTEND_URL=https://your-domain.com
   ```

2. **Enable webhook configuration** if using triggers:
   - Set `AP_FRONTEND_URL` to your public domain
   - Ensure your server is accessible from the internet

3. **Review security settings**:
   - Change default passwords in `.env`
   - Consider enabling sandboxed execution mode
   - Set up SSL/TLS certificates

4. **Configure execution mode** (optional):
   ```
   AP_EXECUTION_MODE=SANDBOX_PROCESS
   ```
   If using `SANDBOX_PROCESS`, uncomment `privileged: true` in the docker-compose file.

## Management Commands

### Rebuild after code changes
```bash
docker compose -f docker-compose.local.yml up --build -d
```

### Stop services
```bash
docker compose -f docker-compose.local.yml down
```

### Stop and remove volumes (⚠️ deletes all data)
```bash
docker compose -f docker-compose.local.yml down -v
```

### Restart a specific service
```bash
docker compose -f docker-compose.local.yml restart activepieces
```

### Execute commands inside the container
```bash
docker compose -f docker-compose.local.yml exec activepieces sh
```

## Volumes

The setup creates the following persistent volumes:

- `postgres_data`: PostgreSQL database files
- `redis_data`: Redis persistence files
- `./cache`: Application cache (mounted from host)

## Networking

All services run on the `activepieces` bridge network, allowing them to communicate internally:

- **activepieces**: Accessible on host port 8080
- **postgres**: Internal port 5432
- **redis**: Internal port 6379

## Troubleshooting

### Container fails to start

Check logs for errors:
```bash
docker compose -f docker-compose.local.yml logs activepieces
```

### Database connection issues

Verify PostgreSQL is running:
```bash
docker compose -f docker-compose.local.yml exec postgres pg_isready -U postgres
```

### Redis connection issues

Test Redis connectivity:
```bash
docker compose -f docker-compose.local.yml exec redis redis-cli ping
```

### Build fails

Clear Docker build cache and rebuild:
```bash
docker builder prune -f
docker compose -f docker-compose.local.yml build --no-cache
```

### Out of disk space

Remove unused Docker resources:
```bash
docker system prune -a --volumes
```

## Advanced Configuration

### Custom PostgreSQL Configuration

Mount a custom postgresql.conf:
```yaml
postgres:
  volumes:
    - ./postgresql.conf:/etc/postgresql/postgresql.conf
```

### Custom Redis Configuration

Mount a custom redis.conf:
```yaml
redis:
  command: redis-server /usr/local/etc/redis/redis.conf
  volumes:
    - ./redis.conf:/usr/local/etc/redis/redis.conf
```

### Enable PM2 for multiple instances

Set in `.env`:
```
AP_PM2_ENABLED=true
```

## Updating

To update Activepieces after pulling new code:

```bash
git pull
docker compose -f docker-compose.local.yml up --build -d
```

## Support

For issues and support:
- GitHub Issues: https://github.com/activepieces/activepieces/issues
- Documentation: https://www.activepieces.com/docs
- Community: https://discord.gg/activepieces
